---
layout: post
title: Introducing Whisper
tags: java whisper product
author: Karthik Abram
---

Its common these days for Java based projects to use a logger like Logback and it is also very common to have an Email appender delivery error emails to your development or support team. This is not only common, but it is also good practice. The enabling of timely feedback can not only help correct issues in a production environment as soon as they happen, but it also allows you to respond to users of the system who were affected by the issue proactively.

A side-effect of such proactive monitoring that is often overlooked is that a simple email appender can very easily result in a flood of emails if something critical (such as user database) goes offline or if a often-used functionality stops working. Logback has a [DuplicateMessageFilter](http://logback.qos.ch/manual/filters.html#DuplicateMessageFilter) that one can configure to suppress duplicates but it is too simplistic. Once it starts to suppress the message, there doesn't seem to be a mechanism to revive the logging.

### Enter Whisper

[Whisper](https://github.com/eclecticlogic/whisper) is a generic appender written to solve precisely this problem. However, the architecture of Whisper allows you to throttle any other "endpoint" appender - SMTPAppender being one of them. In other words, Whisper is an appender that sits in between your log statement and the final appender. It feeds the final appender as long as a per-message threshold is not breached. Once breached it suppresses all messages until idle rate is achieved. 

Whisper is available via Maven Central:

{% highlight xml linenos  %}

<groupId>com.eclecticlogic</groupId>
<artifactId>whisper</artifactId>
<packaging>jar</packaging>
<version>1.0.3</version>

{% endhighlight %}

The Whisper binary and source code can be downloaded from the [Github page](http://eclecticlogic.github.io/whisper/).

### Getting started

The Whisper code ships with a sample logback configuration (under src/sample/resources). But here is the configuration for use with Logback to handle the most common case of suppressing ERROR messages that are sent via the email appender. 

To configure the Whisper appender, first you must configure two other appenders - the regular email appender for ERROR level logs and a second email appender for sending the suppression Digests when suppression kicks in. 

{% highlight xml linenos  %}

<appender name="errorEmail" class="ch.qos.logback.classic.net.SMTPAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <smtpHost>ADDRESS-OF-YOUR-SMTP-HOST</smtpHost>
    <to>EMAIL-DESTINATION</to>
    <from>SENDER-EMAIL</from>
    <subject>TESTING: %logger{20} - %m</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%date %-5level %logger{35} - %message%n</pattern>
    </layout>
</appender>

<appender name="errorDigest" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>ADDRESS-OF-YOUR-SMTP-HOST</smtpHost>
    <to>EMAIL-DESTINATION</to>
    <from>SENDER-EMAIL</from>
    <subject>%X{whisper.digest.subject}</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%date %-5level %logger{35} - %message%n</pattern>
    </layout>
</appender>

{% endhighlight %}

As you can see Whisper adds a special entry into the Mapped-Diagnostic-Context (MDC) to convey an appropriate email subject for the error digest.

Next we configure the Whisper appender itself.

{% highlight xml linenos %}

<appender name="whisper"
    class="com.eclecticlogic.whisper.logback.WhisperAppender">
    <!-- Filter out non error logs -->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <!-- This is the name of the logging category to use to send out error digests. This is associated with the 
    errorDigest appender. -->
    <digestLoggerName>digest.appender.logger</digestLoggerName>
    <!--  suppressAfter specifies the criteria to enter suppression. The example below says that if 3 errors of the same kind
    are encountered within a 5 minute window, then suppression should kick in. -->
    <suppressAfter>3 in 5 minutes</suppressAfter>
    <!-- expireAfter specifies how much of silence the logger must see between messages before stopping suppression. --> 
    <expireAfter>4 minutes</expireAfter>
    <!-- digestFrequency specifies how often error email digests should be sent containing statistics on messages 
    suppressed -->
    <digestFrequency>20 minutes</digestFrequency>

    <!-- The pass-through appender for the normal case when suppression is not in-force. -->
    <appender-ref ref="errorEmail" />
</appender>


{% endhighlight %}

The units for the `suppressAfter` and `expireAfter` elements can be seconds (s, sec, or secs), minutes (m, min or mins) and hours (h, hr or hrs).  

The digest logger name is then associated with the digestAppender and the whisper appender is included in the list of default appenders.

{% highlight xml linenos %}

<logger name="digest.appender.logger" level="error" additivity="false">
    <appender-ref ref="errorDigest" />
</logger>

<root level="debug">
    <appender-ref ref="whisper" />
    <appender-ref ref="fileAppender" />
</root>

{% endhighlight %}

Whisper has been released under the Apache License 2.0. [Download](https://github.com/eclecticlogic/whisper/tarball/master) the library or [fork](https://github.com/eclecticlogic/whisper/fork) the code and give it a shot. I'd love to hear your feedback.