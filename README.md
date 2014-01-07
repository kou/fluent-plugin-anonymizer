# fluent-plugin-anonymizer [![Build Status](https://travis-ci.org/y-ken/fluent-plugin-anonymizer.png?branch=master)](https://travis-ci.org/y-ken/fluent-plugin-anonymizer)

## Overview

Fluentd filter output plugin to anonymize records with MD5/SHA1/SHA256/SHA384/SHA512 algorithms. This data masking plugin protects privacy data such as ID, email, phone number, IPv4/IPv6 address and so on.

## Installation

`````
### native gem
gem install fluent-plugin-anonymizer

### td-agent gem
/usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-anonymizer
`````

## Tutorial

#### configuration

It is a sample to hash record with sha1 for `user_id`, `member_id` and `mail`. For IP address, auto-detecting IPv4/IPv6 and rounding number with 24bit(IPv4) or 104bit(IPv6) netmask using `ipaddr_mask_keys` and `ipv4_mask_subnet`, `ipv6_mask_subnet` option.

`````
<source>
  type forward
  port 24224
</source>

<match test.message>
  type anonymizer
  sha1_keys         user_id, member_id, mail
  ipaddr_mask_keys  host
  ipv4_mask_subnet  24
  ipv6_mask_subnet  104
  remove_tag_prefix test.
  add_tag_prefix    anonymized.
</match>

<match anonymized.message>
  type stdout
</match>
`````

#### result

`````
$ echo '{"host":"10.102.3.80","member_id":"12345", "mail":"example@example.com"}' | fluent-cat test.message
$ echo '{"host":"2001:db8:0:8d3:0:8a2e:70:7344","member_id":"12345", "mail":"example@example.com"}' | fluent-cat test.message

$ tail -f /var/log/td-agent/td-agent.log
2014-01-06 18:30:21 +0900 anonymized.message: {"host":"10.102.3.0","member_id":"8cb2237d0679ca88db6464eac60da96345513964","mail":"914fec35ce8bfa1a067581032f26b053591ee38a"}
2014-01-06 18:30:22 +0900 anonymized.message: {"host":"2001:db8:0:8d3:0:8a2e::","member_id":"8cb2237d0679ca88db6464eac60da96345513964","mail":"914fec35ce8bfa1a067581032f26b053591ee38a"}
`````

## Parameters

* `md5_keys` `sha1_keys` `sha256_keys` `sha384_keys` `sha512_keys`

Specify which hash algorithm to be used for following one or more keys.

* `hash_salt` (default: none)

This salt affects for `md5_keys` `sha1_keys` `sha256_keys` `sha384_keys` `sha512_keys` settings.  
It is recommend to set a hash salt to prevent rainbow table attacks.


* `ipaddr_mask_keys`
* `ipv4_mask_subnet` (default: 24)
* `ipv6_mask_subnet` (default: 104)

Round number for following one or more keys. It makes easy to aggregate calculation. 

| ipv4_mask_subnet |      input      |    output     |
|------------------|-----------------|---------------|
|               24 | 192.168.200.100 | 192.168.200.0 |
|               16 | 192.168.200.100 | 192.168.0.0   |
|                8 | 192.168.200.100 | 192.0.0.0     |

* include_tag_key (default: false)
* tag_key

Add original tag name into filtered record using SetTagKeyMixin.

* remove_tag_prefix
* remove_tag_suffix
* add_tag_prefix
* add_tag_suffix

set one or more option are required for editing tag name using HandleTagNameMixin.

## Notes

* hashing nested value behavior is compatible with [LogStash::Filters::Anonymize](https://github.com/logstash/logstash/blob/master/lib/logstash/filters/anonymize.rb) does. For further details, please check it out the test code at [test_emit_nest_value](https://github.com/y-ken/fluent-plugin-anonymizer/blob/master/test/plugin/test_out_anonymizer.rb#L98).

## Blog Articles

* http://y-ken.hatenablog.com/entry/fluent-plugin-anonymizer-has-released

## TODO

Pull requests are very welcome!!

## Copyright

Copyright © 2013- Kentaro Yoshida ([@yoshi_ken](https://twitter.com/yoshi_ken))

## License

Apache License, Version 2.0
