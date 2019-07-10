# Unnecessary alerts in Alert Manager

Inhibition is a concept of suppressing notifications for certain alerts if certain other alerts are already firing. Inhibitions are configured through the Alertmanager's configuration file.

<<<<<<< HEAD
Some unnecessary alerts are fired always like `WatchDog` (previously `DeadMansSwitch`).`CPUThrottlingHigh` is also one of the unnecessay alerts that are fired due to misconfiguration. These alerts can be troublesome if these alerts are forwarded to a reciever such as Slack etc. and will be fired from time to time producing unnecessary alerts on Slack channel. 
=======
It is seen that some unnecessary alerts are fired always like `WatchDog` (previously `DeadMansSwitch`).`CPUThrottlingHigh` is also one of the unnecessay alerts that are fired due to misconfiguration. These alerts can be troublesome if these alerts are forwarded to a reciever such as Slack etc. and will be fired from time to time producing unnecessary alerts on Slack channel. 
>>>>>>> d53a9afbe866aae24873f03285bfa171082b6c83

To suppress these alerts use `inhibit_rules`. The following inhibit rule is a hack from StackOverflow [here](https://stackoverflow.com/questions/54806336/how-to-silence-prometheus-alertmanager-using-config-files/54814033#54814033)

Inhibit rules on the premise of Watchdog (which is always firing) so those rules will always get inhibited.

```
inhibit_rules:
  - target_match:
      alertname: 'CPUThrottlingHigh'
    source_match:
      alertname: 'Watchdog'
    equal: ['prometheus']
```

And for suppressing Watchdog, create a reciever `alerts-null` and route Watchdog alert to `alerts-null`

```
routes:
- match:
    alertname: Watchdog
  receiver: alerts-null
```