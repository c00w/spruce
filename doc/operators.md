## What are all the Spruce operators?

- [calc](#-calc-)
- [cartesian-product](#-cartesian-product-)
- [concat](#-concat-)
- [defer](#-defer-)
- [empty](#-empty-)
- [file](#-file-)
- [grab](#-grab-)
- [inject](#-inject-)
- [ips](#-ips-)
- [join](#-join-)
- [keys](#-keys-)
- [load](#-load-)
- [negate](#-negate-)
- [param](#-param-)
- [prune](#-prune-)
- [shuffle](#-shuffle-)
- [sort](#-sort-)
- [static_ips](#-static_ips-)
- [stringify](#-stringify-)
- [vault](#-vault-)
- [awsparam](#-awsparam-)
- [awssecret](#-awssecret-)
- [base64](#-base64-)

Additionally, there are operators that are specific to merging arrays. For more detail
see the [array merging documentation][array-merging]:

- `(( append ))` - Adds the data to the end of the corresponding array in the root document.
- `(( prepend ))` - Inserts the data at the beginning of the corresponding array in the root document.
- `(( insert ))` - Inserts the data before or after a specified index, or object.
- `(( merge ))` - Merges the data on top of an existing array based on a common key. This
  requires each element to be an object, all with the common key used for merging.
- `(( inline ))` - Merges the data on top of an existing array, based on the indices of the
  array.
- `(( replace ))` - Removes the existing array, and replaces it with the new one.
- `(( delete ))` - Deletes data at a specific index, or objects identified by the value
  of a specified key.

*Please note:* You cannot use the convenient Spruce path syntax
(`path.to.your.property`) in case one of the elements (e.g. named entry
element) contains a dot as part of the actual key. The dot is in line with the
YAML syntax, however it cannot be used since Spruce uses it as a separator
internally. This also applies to operators, where it is not immediately obvious
that a path is used like with the `(( prune ))` operator. As a workaround,
depending on the actual use-case, it is often possible to replace the Spruce
operator with a equivalent [go-patch] operator file.

## Operator Arguments

Most `spruce` operators have arguments. There are three basic types to the arguments -
literal values (strings/numbers/booleans), references (paths defining a datastructure in
 the root document), and environment variables. Arguments can also make use of a logical-or
(`||`) to failover to other values. See our notes on [environment variables and default values][env-var]
for more information on environment variables and the logical-or.

## (( calc ))

Usage: `(( calc EXPRESSION ))`

The `(( calc ))` operator allows you to perform mathematical operations inside your YAML.
You can reference other values in the data structure, as well as literal numerics. If you
have more sophisticated calculations in mind, you can use these built-in functions inside
your expressions: `max`, `min`, `mod`, `pow`, `sqrt`, `floor`, and `ceil`.

The expression passed to `(( calc ))` must however be a quoted string.

[Example][calc-example]

## (( cartesian-product ))

Usage: `(( cartesian-product LITERAL|REFERENCE ... ))`

The `(( cartesian-product ))` operator accepts an arbitrary number of arguments, outputting
a list made up of the product of all its inputs. If the input is a literal, it is treated as
a one-element set. If it is a reference, it must either be reference a literal type, or an
array filled with only literal types.

[Example][cartesian-example]

## (( concat ))

Usage: `(( concat LITERAL|REFERENCE ... ))`

The `(( concat ))` operator has a role that may shock you. It concatenates values together into
a string. You can pass it any number of arguments, literal or reference, as long as the reference
is not an array/map.

[Example][concat-example]

## (( defer ))

Usage: `(( defer ... ))`

Ever wanted to use `spruce` to generate something with a `spruce` operator
in it, or perhaps something that looks like a spruce operator, like
a CredHub value? Defer can be used for that. When evaluated, it outputs
the contents of the operator, minus the initial defer operator.

[Example][defer-example]

## (( empty ))

Usage: `(( empty hash|map|array|list|string ))`

This operator empties out the contents of the parent. Due to `spruce`'s merging
semantics, it can be a little tricky sometimes to overwrite an array with
an empty aray, or a map with an empty map. Use this as a handy shortcut.

[Example][empty-example]

## (( file ))

Usage: `(( file LITERAL|REFERENCE ))`

Do you have a really long multi-line text file that you want to embed in your YAML?
Perhaps it's an SSL cert, or a base64 encoded image? Of course you do! Enter the
 `(( file ))` operator. Instead of having to worry about the proper YAML indenting
 + multiline semantics when pasting your cert into your file, you can stick the contents
in a file by itself, and provide a path to that file (absolute, or relative to where
`spruce` is executed). The path provided can even be a reference to another part of the data
structure, which might concat some values together to dynamically generate where the file
will be.

**Example:**

```
$ cat <<EOF ca-prod.crt
-----BEGIN CERTIFICATE-----
cert-data
will-go
in-here
-----END CERTIFICATE-----
EOF

$ cat <<EOF > config.yml
files:
  ca: (( concat "ca-" environment ".crt")

environment: prod

load_ca_from_file_ref: (( file files.ca ))
load_ca_from_file: (( file "ca-prod.crt" ))
EOF

$ spruce merge config.yml
environment: prod
files:
  ca: ca-prod.crt
load_ca_from_file: |
  -----BEGIN CERTIFICATE-----
  cert-data
  will-go
  in-here
  -----END CERTIFICATE-----
load_ca_from_file_ref: |
  -----BEGIN CERTIFICATE-----
  cert-data
  will-go
  in-here
  -----END CERTIFICATE-----
```

## (( grab ))

Usage: `(( grab LITERAL|REFERENCE ))`

Trying to DRY up your config file, so that you don't have to change the same property
15 times? `(( grab ))` can help! It pulls the contents of whatever reference you give
it, and stores them as the value of the key it's assigned to. You can also pass
literal values to `(( grab ))`, mostly so you can have default values if desired,
but it's entirely possible to simply `username: (( grab "admin" ))`. I have no idea why you
might want to do that though, since it's much more typing than `username: admin`...

[Example][grab-example]

## (( inject ))

Usage: `(( inject REFERENCE ))`

This works a bit like `(( grab ))`, except that the contents of what it retrieves
are placed at the same level as the key which called the `(( inject ))` operator.
The key containing the `(( inject ))` operator is then removed. In many cases, you
probably want to use `(( grab ))` instead, as it is much more intuitive and easy
to troubleshoot. However, if you want to inject a bunch of data, but override
parts of the data being injected on a case by case basis, this operator will be helpful.

[Example][inject-example]

## (( join ))

Usage: `(( join LITERAL|REFERENCE ... ))`

Sure, `(( concat ))` is great, but what if I have a list of strings that I want as
a single line? Like a users list, authorities, or similar. Do I have to `concat` that
piece by piece? Nope, you can use `join` to concatenate a list into one entry.

[Example][join-example]

## (( keys ))

Usage: `(( keys REFERENCE ))`

Do you need to generate a list containing all the keys of part of your datastructure?
Enter `(( keys ))`. Pass it a reference to part of your datastructure that is a hash/map,
and it will return an array of all of the keys inside it.

[Example][keys-example]

## (( load ))

Usage: `(( load LITERAL|REFERENCE ))`

Similar to the `(( file ))` operator, this operator takes the content of another
file to insert it into the main tree structure. However, `(( load ))` does not
use the content as-is, but expects to parse valid YAML (or JSON). Like the
`(( file ))` operator, you do not have to worry about indentation as Spruce will
cover that for you.

_Note:_ Spruce will **not** evaluate any Spruce operators that might be in
the file that is loaded, because any path used by `grab` or similar would be
ambigious in respect to the document root to be used. If you need to load a
file with Spruce operators in it, you have to run a pre-processing step to
evaluate the file first with another Spruce run.

**Example:**
```
$ cat <<EOF >list.yml
---
- one
- two
...
EOF

$ cat <<EOF >config.yml
---
list: (( load "list.yml" ))
...
EOF

$ spruce merge config.yml
list:
- one
- two

```

## (( negate ))

Usage: `(( negate LITERAL|REFERENCE ))`

This operator takes a bool and negates it: this allows you to DRY up your config file if
you have properties which are inversely related.  As with `(( grab ))`, you can specify
literal values (but why would you?).

## (( param ))

Usage: `(( param LITERAL ))`

Sometimes, you may want to start with a good starting-point
template, but require other YAML files to provide certain values.
Parameters to the rescue!

[Example][param-example]

## (( prune ))

Usage: `(( prune ))`

If you have a need to force the cleanup of data from the final output, but don't want
to rely on the end-user always specifying the necessary `--prune` flags, you can
make use `(( prune ))`s to clear out the bloated data. _Please note:_ Both the CLI
flag as well as the operator will not work if one path element contains a dot as the
actual name, e.g. `10.local: (( prune ))` will **not** work. You have to use the
go-patch equivalent instruction instead.

[Example][prune-example]

## (( shuffle ))

Usage: `(( shuffle LITERAL | REFERENCE [other [args]] ))`

This operator flattens all of its arguments once, into a single
list, and then shuffles them randomly.  This is useful for
switching the order of availability zones for a first
approximation for AZ load balancing.

## (( sort ))

Usage: `(( sort [by KEY] ))`

This operator enables sorting simple lists like lists of strings, or
numbers as well as lists of maps that follow the known contract of containing an
identifying entry each, like `name`, `key`, or `id`. As always, `name` is the
default for named-entry lists if no sort key is specified. The `(( sort ))`
operator works similar to the `(( prune ))` operator as more of an annotation
than an actual operator that immediately performs an action: The path at which
the sort operator is used will be marked for evaluation in the post-processing
phase. That means the sorting will only take place once after all files are
merged.

The `(( sort ))` operator will fail in case of:
- lists that do not contain strings, numbers or maps (for example lists of lists)
- inhomogeneous types (mixing strings and numbers)
- named-entry maps that do not have a single identifying entry


## (( static_ips ))

Usage: `(( static_ips INTEGER ... ))`

Even with BOSH Links, and Cloud Config, it's still occsionally necessary to have static IPs
in your manifest. This operator makes the IP calculation fairly easy, and should be familiar
to anyone who has used `spiff` to do this in the past. You give `(( static_ips ))` a list of
indexes. `spruce` will look through the root document, and find the relevant IP ranges for
static IPs for the network of a VM, and pull in as many as are needed based on the instance
count. It even supports BOSH AZs fairly well.

[Example][static_ips-example]

## (( stringify ))

Usage: `(( stringify REFERENCE ))`

There are use cases, especially with Kubernetes resources like config maps, where one needs to place a piece of a YAML structure as a multiline string. If you happen to have the actual data already in your current file, you can avoid duplicating the content by simply referencing the part of the YAML and the `(( stringify ... ))` operator correctly marshals the data into your tree structure.

[Example][stringify-example]

## (( ips ))

Usage: `(( ips IP_OR_CIDR INDEX [COUNT] ))`

Sometimes you need to reference IP addresses that aren't managed by BOSH, so static_ips
aren't much help. `ips` has you covered. It lets you perform simple addition on IP
addresses. Pass it an IP and an index, and it will add INDEX to the IP. If you pass a
CIDR instead of an IP, the calculation starts from the start of the network. I.e. (( ips
"10.0.0.10/24" 2 )) will yield "10.0.0.2". (( ips "10.0.0.10" 2 )) will yield "10.0.0.12".
If you also specify COUNT, you get a list of IP's instead.
A negative index and an IP will count backwards. A negative index and a CIDR will start
from the end of the given network.

[Example][ips-example]

## (( vault ))

Usage: `(( vault LITERAL|REFERENCE ... ))`

Have sensitive material in your manifests that you don't want stored in the repo that your
configs are in? What do you mean 'No'? Everybody does. The `(( vault ))` operator lets you
store that data in [Vault][vault], and `spruce` will retrieve it at merge time. Simply
specify a vault path in the `secret` backend as the argument, and away it goes. If needed,
you can pull in references to concatenate with info, resulting in an easy way to dynamically
look up Vault paths.

[Example][vault-example]

## (( awsparam ))

Usage: `(( awsparam LITERAL|REFERENCE ... ))`

The `(( awsparam ))` operator will let you pull a value from [AWS SSM Parameter Store](awsparamstore)
at merge time. Specify the parameter store path in one or more arguments that will be joined
to form the whole path and spruce will fetch it for you. Optionally you may pass `?key=...`
to extract a sub-key where the parameter store value is valid JSON or YAML.

[Example][awsparam-example]

## (( awssecret ))

Usage: `(( awssecret LITERAL|REFERENCE ... ))`

The `(( awssecret ))` operator will let you pull a value from [AWS Secrets Manager](awssecretsmanager)
at merge time. Specify the secret name or ARN in one or more arguments that will be joined to
form the whole identifier and spruce will fetch it for you. Optionally you may specify a sub-key
to extract with `?key=...` where the secret value is valid JSON or YAML and either a stage or
version with `?stage=...` / `?version=...` respectively to fetch a specific stage or version.

You may combine these additional arguments with `&`; for example `secret/name?key=subkey&stage=AWSPREVIOUS`.

[Example][awssecret-example]

## (( base64 ))

Usage: `(( base64 LITERAL|REFERENCE ... ))`

When working with configs that require lots of base64 encoding, it can be quite
tedious to manually encode and decode the values when trying to inspect or
update the existing configuration. The `((base64))` operator allows for merge
time base64 encoding of string literals specified directly, or by reference.

[Example][base64-example]

[array-merging]:      https://github.com/geofffranks/spruce/blob/master/doc/array-merging.md
[env-var]:            https://github.com/geofffranks/spruce/blob/master/doc/environment-variables-and-defaults.md
[vault]:              https://vaultproject.io
[go-patch]:           https://github.com/cppforlife/go-patch
[awsparamstore]:      https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html
[awssecretsmanager]:  https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html

[calc-example]:       http://play.spruce.cf/#537ceec949163403ff42fc52331d2c26
[cartesian-example]:  http://play.spruce.cf/#a1bb0cde87c2787b0a46603f3263a70d
[concat-example]:     http://play.spruce.cf/#1420db7abb3e0b39d55e9f6a6dc9c1b4
[defer-example]:      http://play.spruce.cf/#a152b838a0d5fa604a0fd3025127f56b
[empty-example]:      http://play.spruce.cf/#e56b31547de342db18d3283f45301620
[grab-example]:       http://play.spruce.cf/#31673047fdc3f28674c25c42b06b96c7
[inject-example]:     http://play.spruce.cf/#ff8cc8c76b7d54a5d0fcdc2ea0b1d5f8
[join-example]:       http://play.spruce.cf/#0d729640d8dc936d89d2a76d490bcb34
[keys-example]:       http://play.spruce.cf/#b3da7f17c25b1e81799a6ee63a260be8
[param-example]:      http://play.spruce.cf/#b7944defbd5d987c70c25fcbae1756a8
[prune-example]:      http://play.spruce.cf/#ce52f99a0c7470aa2a1e8fd4dddbafff
[static_ips-example]: http://play.spruce.cf/#ce52f99a0c7470aa2a1e8fd4dddbafff
[stringify-example]:  https://play.spruce.cf/#f302027e6a6d6f77c04437d18a420db0
[vault-example]:      https://github.com/geofffranks/spruce/blob/master/doc/pulling-creds-from-vault.md
[ips-example]:        https://spruce.cf/#568526af82aec5448ddf34740dbd70a3
[awsparam-example]:   values-from-aws-parameter-store.md
[awssecret-example]:  values-from-aws-secrets-manager.md
[base64-example]:     https://spruce.cf/#0aa8626b70fd5757fd148d7da4ffec37?update/with/new/version/when/released
