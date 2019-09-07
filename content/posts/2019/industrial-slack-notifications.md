+++
title = "Industrial Slack Notifications"
date = "2019-09-06"
cover = "/assets/2019/slack_notification.jpg"
draft = false
+++

I work on the east coast and get a fair number of inquiries from coworkers after the end of my day, not always seeing the notification on my phone. I don't feel obligated to respond outside of work hours but I do want to know about it so I have the option of responding. I picked up an [industrial stack light](https://www.amazon.com/gp/product/B0746FV12C/) from Amazon for $13.79 which was just the ticket.

I went with a 110V AC model that flashes the light and pulses the buzzer itself so I could cheat and just use z-wave for triggering it, as I had a spare outlet module on hand.

Next we somehow need to know when to turn it on. I wrote a simple [daemon](https://github.com/mikegrb/slackwatch) in Go called slackwatch that takes care of this.

``` go
	config := slackwatch.Config{
		SlackToken: "xoxp-123",
		Actions: []slackwatch.Action{
			slackwatch.CommandAction{Command: "/usr/bin/afplay", Args: "klaxon.wav"},
			slackwatch.URLAction{
				URL:         "https://hassio.lan/api/services/homeassistant/turn_on?api_password=letmein",
				Body:        "{\"entity_id\":\"switch.bat_signal\"}",
				ContentType: "application/javascript",
			},
		},
	}

	s := slackwatch.New(config)
	s.Run()
```

The important bit is the `Actions` field in the `slackwatch.Config` struct which is a slice of things that meet the simple, one method, `slackwatch.Action` interface. When something that deserves a notification happens, the `slackwatch.Message` struct is passed to the execute method.
``` go
type Action interface {
	Execute(Message)
}
```

This makes slackwatch incredibly customizable as arbitrary actions can be created that perform any action you can accomplish in Go. Want to log all notifications to a DB? No problem. Send a tweet when you are DMed? Slackwatch is for you. I've implemented two default actions. One for running a local command, and another for making an HTTP GET or POST request. These should handle the majority of use cases though I'm certainly open to creating, or even better, merging a pull request, for additional default actions that make sense.

For those that don't need any custom actions, a binary is provided that reads a configuration from a json formatted config file in the users' home directory.

``` json
{
  "SlackToken": "xoxp-123",
  "Actions": [
    { "Command": "/usr/bin/afplay", "Args": "klaxon.wav" },
    {
        "URL": "https://hassio.lan/api/services/homeassistant/turn_on?api_password=letmein",
        "Body": "{\"entity_id\":\"switch.bat_signal\"}",
        "ContentType": "application/javascript"
    }
  ]
}
```

Slackwatch also supports a [series of commands](https://github.com/mikegrb/slackwatch/blob/master/pkg/slackwatch/commands.go)
for interacting with it while running, similar to an IRC bot.

I trigger my signal light by making an HTTP post to [Home Assistant](https://www.home-assistant.io), an open source home automation platform. This turns on my signal light via the z-wave network. Home Assistant makes triggering the signal light easy. Turning it off automatically after 10 seconds is just as easy with a simple state based trigger.

``` yaml
  - alias: Bat Signal Timeout
    initial_state: on
    trigger:
      platform: state
      entity_id: switch.bat_signal
      to: 'on'
      for: '00:00:10'
    action:
      - service: switch.turn_off
        data:
          entity_id:
            - switch.bat_signal

```

The full documentation for slackwatch is available on [GoDoc](https://godoc.org/github.com/mikegrb/slackwatch/pkg/slackwatch), and the source is on [GitHub](https://github.com/mikegrb/slackwatch)