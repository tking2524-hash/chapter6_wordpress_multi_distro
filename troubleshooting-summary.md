# Controlled-Failure Troubleshooting Summary

## Temporary Error Introduced
For the required controlled failure, temporarily change the Red Hat PHP MySQL package in `roles/stack_install/tasks/main.yml` from:

```yaml
- php-mysqlnd
```

to:

```yaml
- php-mysql
```

Run the playbook against both hosts and save the output:

```bash
ansible-playbook site.yml --ask-become-pass | tee output/controlled-failure.txt
```

## Expected Result
The Debian host should continue using its own `php-mysql` package list. The Red Hat host should fail if the temporary package name is unavailable on that platform. This demonstrates that package naming is distribution-specific even when the software purpose is the same.

## Correction
Restore the Red Hat package name to:

```yaml
- php-mysqlnd
```

Then rerun the playbook:

```bash
ansible-playbook site.yml --ask-become-pass | tee output/post-fix-success.txt
```

## Final Summary
I intentionally changed a Red Hat-specific PHP package name to a Debian-style package name. The error affected the Red Hat system because the two distributions use different package naming conventions. I corrected the Red Hat package variable and reran the playbook. After the correction, both hosts completed successfully with no failed or unreachable tasks.

> Replace the expected-result wording above with the exact error from your lab output before submitting.
