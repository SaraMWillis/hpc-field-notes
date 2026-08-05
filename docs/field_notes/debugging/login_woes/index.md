# Debugging

## Login Woes

### Tracking down issues in `/var/log/secure`

Repeated messages saying `System Error` with code `4`:
```
pam_sss(sshd:auth): received for user <user>: 4 (System error)
error: PAM: System error for <user> from <host>
```
may artificially inflate a user's failed login tally. Typically is a byproduct of a system issue. An `sss_reset` + tally reset helps. 

From October 6, 2025:

> Pam system error 4 is generally caused by one of the AD servers used for authentication being hosed, and linux’s sssd doesn’t see that in many cases, and continues to attempts against that same server.  