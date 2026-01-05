# ⚠️ THIS REPO IS DEPRECATED ⚠️

> [!WARNING]
> **Hi there, I've deprecated this repo in favor of:**  
> <https://github.com/shokinn/routeros-scripts-custom>
>
> You can find the updated script for the new Hetzner Cloud API there.  
> It also seamlessly integrates with [Christian Hesse](https://github.com/eworm-de)'s [RouterOS Scripts](https://github.com/eworm-de/routeros-scripts), which come a handy update mechanism.

---
---
---

# hetzner-ddns-for-mikrotik

This Mikrotik RouterOS **7** script for updating DNS entries via Hetzner's DNS API.

The script is currently only compatible with **RouterOS 7**!  
RouterOS 6 is currently not supported.

## Table of Contents

- [⚠️ THIS REPO IS DEPRECATED ⚠️](#️-this-repo-is-deprecated-️)
- [hetzner-ddns-for-mikrotik](#hetzner-ddns-for-mikrotik)
  - [Table of Contents](#table-of-contents)
  - [How does is works?](#how-does-is-works)
  - [Setup](#setup)
    - [Script configuration data](#script-configuration-data)
      - [`domainEntryConfig` array data sheet](#domainentryconfig-array-data-sheet)

## How does is works?

The scripts checks the defined interfaces' IP's for the configured [FQDN's](https://en.wikipedia.org/wiki/Fully_qualified_domain_name).  
This is achieved via plain DNS.

If the IP from the interface differs from the IP configures in the DNS record, the DNS record will be updated accordingly to the interfaces' IP.

## Setup

1. Dependencies
   1. This script requires [Winand](https://github.com/Winand)'s [mikrotik-json-parser](https://github.com/Winand/mikrotik-json-parser).
2. Create a new Script `System -> Scripts`
   1. Name: `ddns-hetzner`
   2. Policy: `read`, `write`, `test`, uncheck everything else
   3. Source: Copy the script here
3. Create a [API token for Hetzner's DNS API](https://dns.hetzner.com/settings/api-token)
4. Configure the script to your needs, check the description in the script or below for information how to configure it
5. Create another new script
   1. Name: `JParseFunctions`
   2. Policy: `read`, `write`, `test` uncheck everything else
   3. Source: The content of [mikrotik-json-parser](https://github.com/Winand/mikrotik-json-parser/blob/master/JParseFunctions)
6. Create a new Schedule `System -> Schedule`
   1. Name: `ddns-hetzner`
   2. Start Date: leave it as it is
   3. Start Time: leave it as it is
   4. Interval: `00:05:00`
   5. Policy: `read`, `write`, `test` uncheck everything else
   6. On Event: `ddns-hetzner`

### Script configuration data

|       Variable name |       Data type       | Example                                                                                                                                | Description                                                                                                                                                                           |
| ------------------: | :-------------------: | :------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|            `apiKey` |       `string`        | `:local apiKey "3su1OLc0gUhUdwxn1bmKFss5V19mBhBx";`                                                                                    | This variable requires a valid API token for the [Hetzner DNS API](https://dns.hetzner.com/api-docs). You can create an [API token here](https://dns.hetzner.com/settings/api-token). |
| `domainEntryConfig` | `array`s of `string`s | `:local domainEntryConfig {{"pppoe-out1";"";"domain.com";"A";"@";"300";};{"pppoe-out1";"pool-ipv6";"domain.com";"AAAA";"@";"300";};};` | See below how to format the arrays correctly.                                                                                                                                         |

#### `domainEntryConfig` array data sheet

The `domainEntryConfig` array consists of multiple arrays. Each of the is configuring a DNS record for a given domain in a zone.

The data sheet below describes the formatting of the DNS records arrays.

| Array index | Data          | Data type | Example        | Description                                                                                                                                                              |
| ----------: | :------------ | :-------- | :------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|         `0` | `interface`   | `string`  | `"pppoe-out1"` | Name of the interface where the IP which is currently configured is fetched from.                                                                                        |
|         `1` | `pool`        | `string`  | `"pool-ipv6"`  | The prefix delegation pool which is used to automatically setup the IPv6 interface IP. Use "" when you don't use a pool to set your interface ip or for a type A record. |
|         `2` | `zone`        | `string`  | `"domain.com"` | Zone which should be used to set a record to.                                                                                                                            |
|         `3` | `record type` | `string`  | `"A"`          | Valid values `A`, `AAAA`. The type of record which will be set. Also determines which IP (v4/v6) will be fetched.                                                        |
|         `4` | `record name` | `string`  | `"@"`          | The record name which should be updated. Use `@` for the root of your domain.                                                                                            |
|         `5` | `record TTL`  | `string`  | `"300"`        | TTL value of the record in seconds, for a dynamic entry a short lifetime like 300 is recommended.                                                                        |

Configuration example:

```
:local domainEntryConfig {
    {
        "pppoe-out1";
        "";
        "domain.com";
        "A";
        "@";
        "300";
    };
    {"pppoe-out1";"pool-ipv6";"domain.com";"AAAA";"@";"300";};
    {"pppoe-out1";"";"example.de";"A";"ddns";"300";};
    {"pppoe-out1";"pool-ipv6";"abc.xzy";"AAAA";"ddns";"300";};
};
```

This example will create & update those DNS records:
- domain.com
  - IPv4
  - IPv6
- example.de
  - IPv4
- abc.xzy
  - IPv6
