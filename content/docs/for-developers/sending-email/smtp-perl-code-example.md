---
layout: page
weight: 0
title: SMTP Perl Code Example
group: x-smtpapi
navigation:
  show: true
---

<call-out type="warning">

Categories and Unique Arguments will be stored as a “Not PII” field and may be used for counting or other operations as SendGrid runs its systems. These fields generally cannot be redacted or removed. You should take care not to place PII in this field. SendGrid does not treat this data as PII, and its value may be visible to SendGrid employees, stored long-term, and may continue to be stored after you’ve left SendGrid’s platform.

</call-out>

## SmtpApiHeader.pm

```perl
#!/usr/bin/perl

# Version 1.0
# Last Updated 6/22/2009
use strict;
package SmtpApiHeader;
use JSON;

sub new
{
my $self = shift;
my @a = ();
$self = { 'data' => { }};
bless($self);
return $self;
}

sub addTo
{
my $self = shift;
my @to = @_;
push(@{$self->{data}->{to}}, @to);
}

sub addSubVal
{
my $self = shift;
my $var = shift;
my @val = @_;

if (!defined($self->{data}->{sub}->{$var}))
{
  $self->{data}->{sub}->{$var} = ();
}
push(@{$self->{data}->{sub}->{$var}}, @val);
}

sub setUniqueArgs
{
my $self = shift;
my $val = shift;
if (ref($val) eq 'HASH')
{
  $self->{data}->{unique_args} = $val;
}
}

sub setCategory
{
my $self = shift;
my $cat = shift;
$self->{data}->{category} = $cat;
}

sub addFilterSetting
{
my $self = shift;
my $filter = shift;
my $setting = shift;
my $val = shift;
if (!defined($self->{data}->{filters}->{$filter}))
{
  $self->{data}->{filters}->{$filter} = {};
}
if (!defined($self->{data}->{filters}->{$filter}->{settings}))
{
  $self->{data}->{filters}->{$filter}->{settings} = {};
}
$self->{data}->{filters}->{$filter}->{settings}->{$setting} = $val;
}

sub asJSON
{
my $self = shift;
my $json = JSON->new;
$json->space_before(1);
$json->space_after(1);
return $json->encode($self->{data});
}

sub as_string
{
my $self = shift;
my $json = $self->asJSON;
$json =~ s/(.{1,72})(\s)/$1\n   /g;
my $str = "X-SMTPAPI: $json";
return $str;
}
```

## Example Perl Usage

```perl
#!/usr/bin/perl
use SmtpApiHeader;

my @receiver = ('kyle','bob','someguy');

my $hdr = SmtpApiHeader->new;

my $time = '1pm';
my $name = 'kyle';

$hdr->addFilterSetting('subscriptiontrack', 'enable', 1);
$hdr->addFilterSetting('twitter', 'enable', 1); #please check the apps available for your current package at https://sendgrid.com/pricing
$hdr->addTo(@receiver);
$hdr->addTo('kyle2');

$hdr->addSubVal('-time-', $time);

$hdr->addSubVal('-name-', $time);
$hdr->setUniqueArgs({'test'=>1, 'foo'=>2});

print $hdr->as_string;

print "\n";
</code>

## 	Full Perl Example
 <p>The following code builds a MIME mail message demonstrating all the portions of the SMTP API protocol. To use this example, you will need to have the following perl modules installed:</p>
<ul class="regular">
  <li>MIME::Entity</li>
  <li>Authen::SASL</li>
  <li>JSON</li>
</ul>
<code>
#!/usr/bin/perl
use strict;
use SmtpApiHeader;
use MIME::Entity;
use Net::SMTP;

my $hdr = SmtpApiHeader->new;

# The list of addresses this message will be sent to
my @toList = ('isaac@example', 'tim@example', 'jose@example');

# The names of the recipients
my @nameList = ('Isaac', 'Tim', 'Jose');

# Another substitution variable
my @timeList = ('4pm', '1pm', '2pm');

# Set all of the above variables
$hdr->addTo(@toList);
$hdr->addSubVal('-name-', @nameList);
$hdr->addSubVal('-time-', @timeList);

# Specify that this is an initial contact message
$hdr->setCategory("initial");

# Enable a text footer and set it
$hdr->addFilterSetting('footer', 'enable', 1);
$hdr->addFilterSetting('footer', "text/plain", "Thank you for your business");

my $from = 'you@yourdomain.com';

# For multiple recipient emails, the 'to' address is irrelevant
my $to = 'example@example.com';
my $plain = <<EOM;
Hello -name-,

Thank you for your interest in our products. We have set up an appointment
to call you at -time- EST to discuss your needs in more detail.

Regards,
Fred
EOM

my $html = <<EOM;
<html>
<head></head>
<body>
<p>Hello -name-,<br />
 Thank you for your interest in our products. We have set up an appointment<br />
 to call you at -time- EST to discuss your needs in more detail.<br />

  Regards,<br />
  Fred<br />
</p>
</body>
</html>
EOM

# Create the MIME message that will be sent. Check out MIME::Entity on CPAN for more details
my $mime = MIME::Entity->build(Type  => 'multipart/alternative' ,

Encoding => '-SUGGEST',
From => $from,
To => $to,
Subject => 'Contact Response for <name> at <time>');

# Add the header
$mime->head->add("X-SMTPAPI", $hdr->asJSON);

# Add body
$mime->attach(Type => 'text/plain',
                  Encoding =>'-SUGGEST',
                  Data => $plain);

$mime->attach(Type => 'text/html',
                  Encoding =>'-SUGGEST',
                  Data => $html);

# Login credentials
my $username = 'example@example.com';
my $api_key = "your_api_key";

# Open a connection to the SendGrid mail server
my $smtp = Net::SMTP->new('smtp.sendgrid.net',
                  Port=> 25,
                  Timeout => 20,
                  Hello => "yourdomain.com");

# Authenticate
$smtp->auth($username, $api_key);

# Send the rest of the SMTP stuff to the server
$smtp->mail($from);
$smtp->to($to);
$smtp->data($mime->stringify);
$smtp->quit();
```
