# CronJob
A CronJob starts one-time Jobs on a repeating schedule.

**A CronJob creates Jobs on a repeating schedule.**

CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a crontab (cron table) file on a Unix system. It runs a Job periodically on a given schedule, written in Cron format.

CronJobs have limitations and idiosyncrasies. For example, in certain circumstances, a single CronJob can create multiple concurrent Jobs. See the limitations below.

When the control plane creates new Jobs and (indirectly) Pods for a CronJob, the .metadata.name of the CronJob is part of the basis for naming those Pods. The name of a CronJob must be a valid DNS subdomain value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a DNS label. Even when the name is a DNS subdomain, the name must be no longer than 52 characters. This is because the CronJob controller will automatically append 11 characters to the name you provide and there is a constraint that the length of a Job name is no more than 63 characters.

**Example: **
This example CronJob manifest prints the current time and a hello message every minute:

apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

## Writing a CronJob spec
**Schedule syntax**

The **.spec.schedule ** field is required. The value of that field follows the Cron syntax:

# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *

For example, 0 3 * * 1 means this task is scheduled to run weekly on a Monday at 3 AM.

The format also includes extended "Vixie cron" step values. As explained in the FreeBSD manual:

Step values can be used in conjunction with ranges. Following a range with /<number> specifies skips of the number's value through the range. For example, 0-23/2 can be used in the hours field to specify command execution every other hour (the alternative in the V7 standard is 0,2,4,6,8,10,12,14,16,18,20,22). Steps are also permitted after an asterisk, so if you want to say "every two hours", just use */2.

**Note:**

A question mark (?) in the schedule has the same meaning as an asterisk *, that is, it stands for any of available value for a given field.
