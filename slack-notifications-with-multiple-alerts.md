# Configuring Slack Alerts for a Team

Most of the teams use slack for their collaborration. And as slack gives support for multiple apps like Jira, Github/Gitlab, Jenkins and the list goes on. One can also configure Proetheus AlertManager to send alerts on slack based on your cluster's state.

One can setup all these apps to send alerts/notifications to different slack channels so that the team can coordinate. We believe in using two slack channels for all of your alerts and notifications other than your team collaboration channels.

1. **{project-name}-dev**

This channel can be used for all the mock or dev apps. The state of your apps or if any of the app goes down, one can configure prometheus to send alerts on this channle. In addition to the apps, one can also add the infra-structure apps like Jenkins, Nginx Controller, etc or your cluster health goes down. 

The same channel can be used for your Jira, or Gitlab/Github related tasks. E.g. if a ticket is created or is marked done, you can configure Jira to send notifications to this channel.

2. **{project-name}-prod**

This channel will only be used for production apps. If those apps go down, alerts will get fired in this channel.

This approach can tidy things up as you will not have to setup different channels for different environments and different tools as that can create confusion. With this, you will have only two channels to concentrate upon for your team progress and status.