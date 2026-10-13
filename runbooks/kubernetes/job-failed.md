# A Job or CronJob fails

Sign: a Job with STATUS Failed, "BackoffLimitExceeded" or "DeadlineExceeded" in its conditions, its pods in Error, or a CronJob whose LAST-RUN is older than its schedule allows.

1. Which one: kubeList jobs — COMPLETIONS and STATUS; kubeList cronjobs — SCHEDULE, SUSPEND, LAST-RUN.
   A failed run of a CronJob is a Job named after it with a time suffix: go on with that Job.
2. A failed Job: kubeGet Job, sections status — the Failed condition's reason says which kind:
   - BackoffLimitExceeded → its pods failed more times than spec.backoffLimit. Read why, not the last
     attempt only: kubeLogs with kind Job and its name, grep "error|exception|fatal"; each attempt's pod
     is in kubeList pods with labelSelector job-name=<job>. kubeDiagnose on one of them for the exit code.
     - 137 with OOMKilled → the runbook `oom-killed`.
     - The pod never started (Pending, ImagePullBackOff) → the runbook `pending` or `image-pull`.
     - restartPolicy OnFailure restarts the container in the same pod: kubeLogs with `previous: true`.
   - DeadlineExceeded → spec.activeDeadlineSeconds ran out: the work is slow, or it waits on something.
     A dependency it cannot reach: kubeProbe from its pod to that host and port.
   - No pods left to read → spec.ttlSecondsAfterFinished or the CronJob's failedJobsHistoryLimit removed
     them: say so, read kubeEvents of the Job, and catch the next run instead of guessing.
3. A CronJob that does not run:
   - SUSPEND true → it was stopped on purpose; kubeFieldHistory on it says when. Resume (kubeSuspend
     with suspend false) only when the user asks.
   - kubeGet CronJob, sections status — status.active still holds a Job and spec.concurrencyPolicy is
     Forbid → every run is skipped while that one hangs: the hanging Job is the problem, step 2.
   - kubeEvents of the CronJob about too many missed start times → the controller gave up; the cause is
     usually a long suspension or a startingDeadlineSeconds too short.
   - Runs at the wrong hour → the schedule is UTC unless spec.timeZone says otherwise.
4. It failed after a change: kubeFieldHistory on the CronJob (jobTemplate: image, command, env) and
   compare with the last run that succeeded (status.lastSuccessfulTime).

Do not: delete the Job or raise backoffLimit to make it pass before the cause is named — it fails the same way.
A run by hand (kubeApply of a Job built from the CronJob's jobTemplate) is a fair offer once the cause is fixed.
