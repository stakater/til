# Setup Domain Verification For G-Suite

I had a domain on AWS and want to setup Domain Verification for G-Suite, for this, you need to add MX records in AWS hosted-zone. You can go to G-Suite, and it will give you option to verifiy through TXT record, A-record or MX record. You can select MX-record. It will give you 6 records. 5 will be generic one as follows:

```txt
"1 ASPMX.L.GOOGLE.COM",
"5 ALT1.ASPMX.L.GOOGLE.COM",
"5 ALT2.ASPMX.L.GOOGLE.COM",
"10 ASPMX2.GOOGLEMAIL.COM",
"10 ASPMX3.GOOGLEMAIL.COM",  
```

One will be specific to it that needs it for verification. Depending upon your domain provider add these records in your hosted-zone. E.g. In AWS, add a record and keep its name empty of type MX and in values add 

```txt
"15 {YOUR_MX_VERIFICATION_RECORD}",
"1 ASPMX.L.GOOGLE.COM",
"5 ALT1.ASPMX.L.GOOGLE.COM",
"5 ALT2.ASPMX.L.GOOGLE.COM",
"10 ASPMX2.GOOGLEMAIL.COM",
"10 ASPMX3.GOOGLEMAIL.COM",
```

If you want to do it with terraform, add the following resource and run `terraform apply`

```tf
resource "aws_route53_record" "example_mx" {
  zone_id = "${data.aws_route53_zone.zone.zone_id}"
  name    = ""
  type    = "MX"
  records = [
    "15 ${var.mx_verification_record}",
    "1 ASPMX.L.GOOGLE.COM",
    "5 ALT1.ASPMX.L.GOOGLE.COM",
    "5 ALT2.ASPMX.L.GOOGLE.COM",
    "10 ASPMX2.GOOGLEMAIL.COM",
    "10 ASPMX3.GOOGLEMAIL.COM",  
  ]
  allow_overwrite = true
  ttl = "3600"
}
```

And then try for Verification again, it should work.
