# IPv6

{{ kops_feature_table(kops_added_ff='1.23') }}

kOps has experimental support for configuring clusters with IPv6-only pods and dual-stack nodes.

IPv6 mode is specified by setting `nonMasqueradeCIDR: "::/0"` in the cluster spec.
The `--ipv6` flag of `kops create cluster` sets this field, among others.

## Cloud providers

kOps currently supports IPv6 on AWS only.

IPv6 requires the external Cloud Controller Manager.

## VPC and subnets

The VPC can be either shared or managed by kOps. If shared, it must have an IPv6 pool associated.

Subnet IPv6 CIDR allocations may be specified in the cluster spec using the special syntax `/LEN#N`,
where "LEN" is the prefix length and "N" is the hexadecimal sequence number of the CIDR within the VPC's IPv6 CIDR.
For example, if the VPC's CIDR is `2001:db8::/56` then the syntax `/64#a` would mean `2001:db8:0:a/64`.

## Routing and NAT64

Managed private and public subnets which have `IPv6CIDR` assignments route `64:ff9b::/96` (NAT64) to whatever is specified in the
`egress` field of the subnet's spec, defaulting the availability zone's NAT Gateway.

If a NAT Gateway is thus needed by a managed public subnet and there are no utility subnets in that availability zone,
the NAT Gateway will be placed in the first-listed public subnet in that zone.

The managed private subnets route the rest of outbound IPv6 traffic to the VPC's Egress-only Internet Gateway.
The managed public subnets route the rest of outbound IPv6 traffic to the VPC's Internet Gateway.

## CNI

kOps currently supports IPv6 on Calico, Cilium, and bring-your-own CNI only.

CNIs must not masquerade IPv6 addresses.

### Calico

Running IPv6 with Calico requires a Debian 11-based AMI. As of the writing of this document, Ubuntu does not work due to an 
[issue with systemd's handling of AWS's incorrect DHCP responses](https://github.com/systemd/systemd/issues/20803).

## Future work

* External-DNS does not, as of the writing of this document, support registering AAAA records.
