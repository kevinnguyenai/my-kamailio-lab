# PrivateDial

PrivateDial is a suite of [Asterisk configuration files](https://wiki.asterisk.org/wiki/display/AST/Asterisk+Configuration+Files). This configuration is tailored to residential use cases, supporting the capabilities of mobile smart phones, that is, voice, video, instant messaging or SMS, and voice mail delivered by email.

It uses the [PJSIP](https://www.pjsip.org/) [channel driver](https://wiki.asterisk.org/wiki/display/AST/Configuring+res_pjsip) and therefore natively support simultaneous connection of several soft-phones to each user account/endpoint.

The underlying design idea is to separate the dial plan functionality from the user data. To achieve this all user specific data has been pushed out from the main `extensions.conf` file.

## Features

Feature list follows below

- Calls and SMS between local endpoints.
- ITSP originating (incoming) SIP voice calls.
- ITSP termination (outgoing) SIP voice call.
- WebSMS; SMS to and from ITSP.
- [MiniVoiceMail](https://wiki.asterisk.org/wiki/display/AST/Asterisk+16+Application_MinivmRecord)


## Configuration files

The suite of Asterisk configuration files making up PrivateDial is summarized below.

### Configuration files overview

The configuration files making up PrivateDial are tabulated below.

| File name             | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| extensions.conf       | The dial plan, defining the data flow of calls and messages  |
| extensions_local.conf | Use case specific global variables used in extensions.conf   |
| minivm.conf           | Define mail sever URL and authentication credentials which voice mail email notifications will be sent |
| pjsip.conf            | Use case specific global variables used by the PJSIP driver  |
| pjsip_transport.conf  | Defines SIP transport, protocol, port, host URL              |
| pjsip_wizard.conf     | Defines templates for sip trunk and soft-phone endpoints     |
| pjsip_endpoint.conf   | Defines sip trunk and soft-phone endpoints                   |

When configuring the asterisk sever the following files often needs to be updated: `pjsip_transport.conf` and `minivm.conf`. The remaining task is, once the severer has been configured, to add and maintain sip trunk and soft-phone endpoints, which is kept in `pjsip_endpoint.conf`.

## Usage

### SIP Trunk

PJSIP endpoints are defined using the [PJSIP Wizard](https://wiki.asterisk.org/wiki/display/AST/PJSIP+Configuration+Wizard) in the configuration file `pjsip_endpoint.conf`. For convenience the template, `w_trunk` has been defined in `pjsip_wizard.conf`.

Add an endpoint entry in `pjsip_endpoint.conf` based on the setup instructions provided by your trunk provider. This entry also hold your authentication credentials.

`pjsip_endpoint.conf`

```ini
[itsp:mydoe](w_trunk)
remote_hosts = sip.mydoe.com
sends_auth = yes
sends_registrations = yes
outbound_auth/username = username
outbound_auth/password = password
```

With some ITSP SIP servers you need to explicitly state which transport to use. In such case add the folowing to the section above; `transport = udp`, for UDP.

### SIP Users

PJSIP endpoints are defined using the [PJSIP Wizard](https://wiki.asterisk.org/wiki/display/AST/PJSIP+Configuration+Wizard) in the configuration file `pjsip_endpoint.conf`. For convenience the template, `w_term_io` and `w_term_i` has been defined in `pjsip_wizard.conf`.

Add an endpoint entry in `pjsip_endpoint.conf` for each user. Each user can simultaneously connect with several soft-phones, using the same account.

`pjsip_endpoint.conf`

```ini
[w_term:mydoe](!,w_term_io)
endpoint/set_var = TRUNK_ENDPOINT=itsp:mydoe
endpoint/set_var = WEBSMS_INDEX=

[john.doe](w_term:mydoe)
hint_exten = +12025550160
endpoint/callerid = John Doe <+12025550160>
endpoint/mailboxes = john.doe@example.com
inbound_auth/username = john.doe
inbound_auth/password = password
```

You also need to configure WebSMS for SMS to work, see separate documentation.

### Outgoing SMTP email server

PrivateDial use [MiniVoiceMail](https://wiki.asterisk.org/wiki/display/AST/Asterisk+16+Application_MinivmRecord) to deliver voice mail messages via email with attached sound files. For this to work a separate SMTP email server need to have been set up. This can for example be achieved by using the image [mlan/postfix-amavis](https://hub.docker.com/repository/docker/mlan/postfix-amavis). With a functional email server, configure MiniVM to connect to it by providing its URL and authentication credentials in `minivm.conf`

`minivm.conf`

```ini
[general]
mailcmd = minivm-send -H mx.example.com:587 -P starttls -u username -p password -f voicemail-noreply@example.com
```

### SIP Networking

Here we describe 3 aspects of SIP networking that often needs to be addressed. Communication with devices on local networks, Intrusion prevention using non-standard ports. Privacy using encryption.

#### Network Address Translation (NAT)

When communicating with devices on local networks a more elaborate mechanism using (NAT) needs to be configured allowing server and client locate each other. Assuming that the SIP server has the following external URL; `sip.example.com`, make sure to update `pjsip_transport.conf` so it includes the snippet below.

`pjsip_transport.conf`

```ini
[t_wan](!)
type = transport
bind = 0.0.0.0:5060
domain = example.com
external_signaling_address = sip.example.com
external_media_address = sip.example.com
```

#### Custom SIP ports

When using non-standard ports the amount of attacks drop significantly, so it might be considered whenever practical. When changing port numbers they need to be updated both for docker and asterisk. To exemplify, assume we want to use 5560 for UDP and TCP and 5561 for TLS, in which case we update the configuration in two places:

- docker/docker-compose, eg, `docker run -p "5560-5561:5560-5561" -p"5560:5560/udp" ...`
- asterisk transport in `pjsip_transport.conf`

`pjsip_transport.conf`

```ini
[t_wan](!)
type = transport
bind = 0.0.0.0:5560
...
[tls](t_wan)
bind = 0.0.0.0:5561
...
```

#### TLS Certificate and key

To enable encryption of both the session and data packages (TLS and SDES SRTP) a [TLS/SSL server certificate](https://en.wikipedia.org/wiki/Public_key_certificate) and key are needed. If the certificate and key do not exist when the container starts a [self-signed certificate](https://en.wikipedia.org/wiki/Self-signed_certificate) and private key are automatically generated. The default file names for these are defined below. Should the certificate and key be available be other means they can be copied to the container using this names. If other file names are referred also update their names in `pjsip_transport.conf`.

`pjsip_transport.conf`

```ini
[tls](t_wan)
cert_file = /etc/ssl/asterisk/cert.pem
priv_key_file = /etc/ssl/asterisk/priv_key.pem
```

There is also a mechanism to use ACME lets encrypt certificates, which also use these file names.

## Implementation

### Dialplan

The PrivateDial has its dialplan contexts organized in 3 levels. The entry, action and subroutine contexts. A SIP event will trigger the execution of the PrivateDial dial plan staring on one of the entry contexts. The entry contexts include some of the action contexts, and the action contexts call the subroutines.

#### Entry context

The entry contexts are used to grant more access to users calling or texting as compared to external trunk calls or texts. All entry context start with including the `dp_lookup` context so that extension hints are always available.

```ini
[dp_entry_call_inout](+)
include => dp_lookup
include => dp_ivr_recgreet
include => dp_call_inout

[dp_entry_call_in](+)
include => dp_lookup
include => dp_call_in

[dp_entry_text_inout](+)
include => dp_lookup
include => dp_text_inout

[dp_entry_text_in](+)
include => dp_lookup
include => dp_text_in

[dp_entry_answer](+)
include => dp_lookup
include => dp_answer
```

#### Action context

The action contexts calls the subroutines. Most subroutines use the `${HINT}` channel variable to identify the endpoint so `${EXTEN}` is set to the special `s`. Each subroutine is called in its turn and the call is not hung up until all subroutine calls has been made.

```ini
[dp_lookup]
; hints are placed here see hint_exten in pjsip_wizard.conf
exten => _0ZXXXXXX.,1,Goto(${CONTEXT},+${GLOBAL(CONTRY_CODE)}${EXTEN:1},1)
exten => _ZXXXXXX.,1,Goto(${CONTEXT},+${EXTEN},1)

[dp_call_inout]
exten => _[+0-9].,1,NoOp()
 same => n,Gosub(sub_dial_term,s,1(${HINT}))
 same => n,Gosub(sub_voicemail,s,1(${HINT}))
 same => n,Gosub(sub_dial_trunk,${EXTEN},1(${HINT}))
 same => n,Hangup()

[dp_call_in]
exten => _[+0-9].,1,NoOp()
 same => n,Gosub(sub_dial_term,s,1(${HINT}))
 same => n,Gosub(sub_voicemail,s,1(${HINT}))
 same => n,Hangup()

[dp_text_inout]
exten => _[+0-9].,1,NoOp()
 same => n,Gosub(sub_rewrite_from,s,1)
 same => n,Gosub(sub_text_term,s,1(${HINT}))
 same => n,Gosub(sub_text_trunk,${EXTEN},1(${HINT}))
 same => n,Hangup()

[dp_text_in]
exten => _[+0-9].,1,NoOp()
 same => n,Gosub(sub_decode_body,s,1)
 same => n,Gosub(sub_text_term,s,1(${HINT}))
 same => n,Hangup()

[dp_answer]
exten => _[+0-9].,1,Goto(dev-${DEVICE_STATE(${HINT})})
 same => n(dev-NOT_INUSE),NoOp()
 same => n(dev-INUSE),NoOp()
 same => n(dev-RINGING),NoOp()
 same => n(dev-RINGINUSE),NoOp()
 same => n(dev-ONHOLD),NoOp()
 same => n(dev-UNAVAILABLE),NoOp()
 same => n,Answer()
 same => n(dev-UNKNOWN),NoOp()
 same => n(dev-INVALID),NoOp()
 same => n,Hangup()
```

#### Subroutine context

The file `extension.conf` include some in line documentation of the subroutines.

Subroutines does not hang up but instead returns the data flow to the calling context. (CHECK with sub_voicemail).

Most subroutines use the `${HINT}` channel variable to identify the endpoint so `${EXTEN}` is set to the special `s`.

When calling and texting endpoints an attempts are made to contact all contacts of the endpoints, such that for inbound calls all registered contacts (smart-pones) will ring and also receive inbound SMS.

### Presence

Asterisk support [presence state](https://wiki.asterisk.org/wiki/display/AST/Presence+State) an indicator that conveys the state of an endpoint to other endpoints. But, unfortunately, presence is only supported for Sangama/Digium phones, not for softphones. Consequentely, no attmpt has been made to support presence state in PrivateDial.
