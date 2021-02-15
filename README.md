# Sealed Secrets Example

Demonstrating how to use ``sealed-secrets`` to encrypt and store Kubernetes secrets in git on a k3s cluster using k3d.

# What Problem does sealed-secrets Solve?

Secrets cannot be managed in git. Sealed Secrets can help encrypt your Secrets into SealedSecret which is safe to store in a public repository. It can be decrypted only by the controller running in the target cluster.

# Install k3d
k3d is a little helper to run k3s in docker, where k3s is the lightweight Kubernetes distribution by Rancher. It actually removes millions of lines of code from k8s. If you just need a learning playground, k3s is definitely your choice.

Check out [k3d Github Page](https://github.com/rancher/k3d#get) to see the installation guide.

> When creating a cluster, ``k3d`` utilises ``kubectl`` and ``kubectl`` is not part of ``k3d``. If you don't have ``kubectl``, please install and set up [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/). 

Once you've installed ``k3d`` and ``kubectl``, run

```
➜  k3d create -n sealed-secrets-example
```

We need to make ``kubectl`` to use the kubeconfig for that cluster.
```
➜  export KUBECONFIG="$(k3d get-kubeconfig --name='sealed-secrets-example')"
```

# Install kubeseal

The kubeseal utility is the first part of Sealed Secrets, and it uses asymmetric crypto to encrypt secrets that only the controller can decrypt.

```
➜  brew install kubeseal
```

# Apply sealed-secrets controller

The second part of Sealed Secrets is a cluster-side controller / operator.

```
➜  kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.14.1/controller.yaml

rolebinding.rbac.authorization.k8s.io/sealed-secrets-controller created
clusterrolebinding.rbac.authorization.k8s.io/sealed-secrets-controller created
serviceaccount/sealed-secrets-controller created
customresourcedefinition.apiextensions.k8s.io/sealedsecrets.bitnami.com created
rolebinding.rbac.authorization.k8s.io/sealed-secrets-service-proxier created
role.rbac.authorization.k8s.io/sealed-secrets-service-proxier created
role.rbac.authorization.k8s.io/sealed-secrets-key-admin created
clusterrole.rbac.authorization.k8s.io/secrets-unsealer created
deployment.apps/sealed-secrets-controller created
service/sealed-secrets-controller created
```

```
➜  kubectl logs --tail=-1 -f -l name=sealed-secrets-controller -n kube-system
```

```
controller version: v0.14.1
2021/02/15 04:24:29 Starting sealed-secrets controller version: v0.14.1
2021/02/15 04:24:29 Searching for existing private keys
2021/02/15 04:24:32 New key written to kube-system/sealed-secrets-keynvpcz
2021/02/15 04:24:32 Certificate is
-----BEGIN CERTIFICATE-----
MIIErjCCApagAwIBAgIRAIh+6dntUQrrqSAWQ4gcwJ8wDQYJKoZIhvcNAQELBQAw
ADAeFw0yMTAyMTUwNDI0MzJaFw0zMTAyMTMwNDI0MzJaMAAwggIiMA0GCSqGSIb3
DQEBAQUAA4ICDwAwggIKAoICAQDxXLMWTnq5Z42APQ6pZZGTAaXS1BsNDOcJuIIE
Yv+bqKS0tb1cTbXDCpPxxqCj35iuI/jLy9LhjqTlzJFtDevUf2F2b2RzvRaEtHIY
pvAXMfNCVTkUMOuTGB72rTLx2KI1mYg6OI+PDh26BrWkxlbW8oFe7knoeXO3n6dJ
j8W9Ypg5ZXNxLqcagk/HmSAoGLQkzaq9rQiVVM2jA+LPUImH4jIph9BYvWjL+cLv
ax0WhybTYk/YAb+/4sV7OMI6AHl4e8jNgrVVj3DdWDhv3sNmuxshk0OypZ2fwrqV
s9N8so+7JADWQnXw1MTOAec4CS6RbVLl3RTwegDszgpZ9jXafy0WH3CNrdeVSJUl
3NJ8Qy40Y1mnrz3Qa+TbGjV9LTn2FpC3g59Zib2mtb62RYy0jOko54MjURtUao/n
p26P1/BtkPN024hpNngbKwN5JluNdeP6KDFiLwsAsYGs/YhyVoUa7vo+iMmHNwVr
FsyaGOEPSPRdi6KXj/HgxYAxMNpYY6b41rvOeQ1paf4c9f6L7NJamKyM+a8R/zt7
F06c7aHJFdvU4M4bMbRqVC7jmRa8y6fPRu/9jSfQ1lR2B1BJgEAGbk149bvbWiZ9
jP+vz5T3qLpWVqv05RTdAfusEDcmImSbrzSU3l2y67nmshAkPoZpM8PqBSVfrA6N
R99tiwIDAQABoyMwITAOBgNVHQ8BAf8EBAMCAAEwDwYDVR0TAQH/BAUwAwEB/zAN
BgkqhkiG9w0BAQsFAAOCAgEAxLMHynW7ZzAEVblRnB8bydlcCG4ddSTghLh085vN
OkzQOLDDaC++vTWzQA6CWyzNNkpuXpp9r3tzy6qcFvYfXSJrT1ZwyzWzgv2K+Bjm
M/OuLtOYLNT4L2SZxIPSc60lUn77RV11I7dFY3UNkqRvH7Gu+qQSYZFU1PZSTt5n
BSXHT2/+GaVD9bdK9/lfF6vNDeut3iEfY9pi3BR8d3G/EDz9B5fS/NPXV8gzXPUf
Bth3bUIZv+a06PNO7t02RarkskN7JIOb1WniYUjWxdYq7PGR6yriItGvLdRzrX9s
wtWrrUfhBSbNJW5CYMjZyxPQDdrh0SGm2aiAUTQux++jS63cehhhjAUSvKiklCEg
i2uuq5VuV1cbyq8gf3jCFXh9d5EjJE4eCMUEamh3NwIc2Tbl4C+KarVNxgFvPL8+
Nmnc8aVgHISuVxxcUz28Q0//I8VZrcsyw0b4T7X5exRRcEk0bW/CzX+Tqrjk3Pep
6FKvQP1iXV5WIocodGoMmOBhhihNjZOWugaaQpk41PCUqMkOnOyXmQ8bLI7E/wvB
VpDKjKQeCRbUAkDmFgRHoT8jYkaEeI43G2kWQTlPACNY6spsabKDw570j8gzHUTn
kEsUTISM/1/lnTpMV3me24VEDOlUFELBp/kXx3QAA0xO5IpHdaIMnse68oVIT9kZ
PBc=
-----END CERTIFICATE-----

2021/02/15 04:24:32 HTTP server serving on :8080
```

# Verify the sealed-secrets controller

```
➜  kubectl get pods -n kube-system
```

```
sealed-secrets-controller-59f9b7b6f4-4qsbq   1/1     Running     1          2m20s
```

```
➜  kubectl get secrets -n kube-system
```

```
sealed-secrets-controller-token-27cjx                kubernetes.io/service-account-token   3      4m29s
sealed-secrets-keynvpcz                              kubernetes.io/tls                     2      3m33s
```

Take a look at the secret ``sealed-secrets-keynvpcz``

```
➜  kubectl get secrets sealed-secrets-keynvpcz -n kube-system -o yaml
```

```yml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVyakNDQXBhZ0F3SUJBZ0lSQUloKzZkbnRVUXJycVNBV1E0Z2N3Sjh3RFFZSktvWklodmNOQVFFTEJRQXcKQURBZUZ3MHlNVEF5TVRVd05ESTBNekphRncwek1UQXlNVE13TkRJME16SmFNQUF3Z2dJaU1BMEdDU3FHU0liMwpEUUVCQVFVQUE0SUNEd0F3Z2dJS0FvSUNBUUR4WExNV1RucTVaNDJBUFE2cFpaR1RBYVhTMUJzTkRPY0p1SUlFCll2K2JxS1MwdGIxY1RiWERDcFB4eHFDajM1aXVJL2pMeTlMaGpxVGx6SkZ0RGV2VWYyRjJiMlJ6dlJhRXRISVkKcHZBWE1mTkNWVGtVTU91VEdCNzJyVEx4MktJMW1ZZzZPSStQRGgyNkJyV2t4bGJXOG9GZTdrbm9lWE8zbjZkSgpqOFc5WXBnNVpYTnhMcWNhZ2svSG1TQW9HTFFremFxOXJRaVZWTTJqQStMUFVJbUg0aklwaDlCWXZXakwrY0x2CmF4MFdoeWJUWWsvWUFiKy80c1Y3T01JNkFIbDRlOGpOZ3JWVmozRGRXRGh2M3NObXV4c2hrME95cFoyZndycVYKczlOOHNvKzdKQURXUW5YdzFNVE9BZWM0Q1M2UmJWTGwzUlR3ZWdEc3pncFo5alhhZnkwV0gzQ05yZGVWU0pVbAozTko4UXk0MFkxbW5yejNRYStUYkdqVjlMVG4yRnBDM2c1OVppYjJtdGI2MlJZeTBqT2tvNTRNalVSdFVhby9uCnAyNlAxL0J0a1BOMDI0aHBObmdiS3dONUpsdU5kZVA2S0RGaUx3c0FzWUdzL1loeVZvVWE3dm8raU1tSE53VnIKRnN5YUdPRVBTUFJkaTZLWGovSGd4WUF4TU5wWVk2YjQxcnZPZVExcGFmNGM5ZjZMN05KYW1LeU0rYThSL3p0NwpGMDZjN2FISkZkdlU0TTRiTWJScVZDN2ptUmE4eTZmUFJ1LzlqU2ZRMWxSMkIxQkpnRUFHYmsxNDlidmJXaVo5CmpQK3Z6NVQzcUxwV1ZxdjA1UlRkQWZ1c0VEY21JbVNicnpTVTNsMnk2N25tc2hBa1BvWnBNOFBxQlNWZnJBNk4KUjk5dGl3SURBUUFCb3lNd0lUQU9CZ05WSFE4QkFmOEVCQU1DQUFFd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTgpCZ2txaGtpRzl3MEJBUXNGQUFPQ0FnRUF4TE1IeW5XN1p6QUVWYmxSbkI4YnlkbGNDRzRkZFNUZ2hMaDA4NXZOCk9relFPTEREYUMrK3ZUV3pRQTZDV3l6Tk5rcHVYcHA5cjN0enk2cWNGdllmWFNKclQxWnd5eld6Z3YySytCam0KTS9PdUx0T1lMTlQ0TDJTWnhJUFNjNjBsVW43N1JWMTFJN2RGWTNVTmtxUnZIN0d1K3FRU1laRlUxUFpTVHQ1bgpCU1hIVDIvK0dhVkQ5YmRLOS9sZkY2dk5EZXV0M2lFZlk5cGkzQlI4ZDNHL0VEejlCNWZTL05QWFY4Z3pYUFVmCkJ0aDNiVUladithMDZQTk83dDAyUmFya3NrTjdKSU9iMVduaVlVald4ZFlxN1BHUjZ5cmlJdEd2TGRSenJYOXMKd3RXcnJVZmhCU2JOSlc1Q1lNalp5eFBRRGRyaDBTR20yYWlBVVRRdXgrK2pTNjNjZWhoaGpBVVN2S2lrbENFZwppMnV1cTVWdVYxY2J5cThnZjNqQ0ZYaDlkNUVqSkU0ZUNNVUVhbWgzTndJYzJUYmw0QytLYXJWTnhnRnZQTDgrCk5tbmM4YVZnSElTdVZ4eGNVejI4UTAvL0k4VlpyY3N5dzBiNFQ3WDVleFJSY0VrMGJXL0N6WCtUcXJqazNQZXAKNkZLdlFQMWlYVjVXSW9jb2RHb01tT0JoaGloTmpaT1d1Z2FhUXBrNDFQQ1VxTWtPbk95WG1ROGJMSTdFL3d2QgpWcERLaktRZUNSYlVBa0RtRmdSSG9UOGpZa2FFZUk0M0cya1dRVGxQQUNOWTZzcHNhYktEdzU3MGo4Z3pIVVRuCmtFc1VUSVNNLzEvbG5UcE1WM21lMjRWRURPbFVGRUxCcC9rWHgzUUFBMHhPNUlwSGRhSU1uc2U2OG9WSVQ5a1oKUEJjPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlKS0FJQkFBS0NBZ0VBOFZ5ekZrNTZ1V2VOZ0QwT3FXV1Jrd0dsMHRRYkRRem5DYmlDQkdML202aWt0TFc5ClhFMjF3d3FUOGNhZ285K1lyaVA0eTh2UzRZNms1Y3lSYlEzcjFIOWhkbTlrYzcwV2hMUnlHS2J3RnpIelFsVTUKRkREcmt4Z2U5cTB5OGRpaU5abUlPamlQanc0ZHVnYTFwTVpXMXZLQlh1NUo2SGx6dDUrblNZL0Z2V0tZT1dWegpjUzZuR29KUHg1a2dLQmkwSk0ycXZhMElsVlROb3dQaXoxQ0poK0l5S1lmUVdMMW95L25DNzJzZEZvY20wMkpQCjJBRy92K0xGZXpqQ09nQjVlSHZJellLMVZZOXczVmc0Yjk3RFpyc2JJWk5Ec3FXZG44SzZsYlBUZkxLUHV5UUEKMWtKMThOVEV6Z0huT0FrdWtXMVM1ZDBVOEhvQTdNNEtXZlkxMm44dEZoOXdqYTNYbFVpVkpkelNmRU11TkdOWgpwNjg5MEd2azJ4bzFmUzA1OWhhUXQ0T2ZXWW05cHJXK3RrV010SXpwS09lREkxRWJWR3FQNTZkdWo5ZndiWkR6CmROdUlhVFo0R3lzRGVTWmJqWFhqK2lneFlpOExBTEdCclAySWNsYUZHdTc2UG9qSmh6Y0ZheGJNbWhqaEQwajAKWFl1aWw0L3g0TVdBTVREYVdHT20rTmE3em5rTmFXbitIUFgraSt6U1dwaXNqUG12RWY4N2V4ZE9uTzJoeVJYYgoxT0RPR3pHMGFsUXU0NWtXdk11bnowYnYvWTBuME5aVWRnZFFTWUJBQm01TmVQVzcyMW9tZll6L3I4K1U5Nmk2ClZsYXI5T1VVM1FIN3JCQTNKaUprbTY4MGxONWRzdXU1NXJJUUpENkdhVFBENmdVbFg2d09qVWZmYllzQ0F3RUEKQVFLQ0FnQWF4eVViV1hPbU5FWHZyMVo4RnNleTNxRHVKaGdtTjRNK2dkank4YVRZT1Rxa3pmRUhWNXZOMnRPVgpKR3RZSXd1R2JubEE2d2tuZXpMeVIrTHVqWGZYcUpaQWxKVTVmZ1lNalJTSGhhWG5mT1EzUE10TFlTNFJzTUJtCnI4cVNLRzIrc3B6NWtLTGt4VFVwR1d0M3I0V2M3V1RMQ25icXN1YlN2WVRLMVllanZsZVRMcDFETm1EVndSVm0KMktkSHE0MzQ4MVI1SE1SeUJPbVhwMnUzZ29EdnNYbk5QOE11eFR6bVBIeVRJWGdsc3JMdEN1QSszOXJOU0RTTwp1anBhUXdrM0E4ekFlRHIwRmlqNGRidzFOU3JLc0FHUGxRNFN1T3NtK1d6SUJSNTJuRHowRDBlRWZmVWwxZ1ZMCnNjeGNYREJ0ZEFxWmRCREpxVStHOWtrUnVBNDdTMjZMSEg0RHhMZUlrcGQ0U3pYajNSa0IybENpeVhCZGpvZGcKaVRsSXNlQkl5OWpTcUltdDUrMS9oMlJKK2pSOVVpSTJuYWtlUDNNRWFHUDNaR0dWd1lFRkQ0UFVpdkhWUHl3aAppTCtaK2dqL3hDSGdrK0x5U2NHMXZXT0I2L2ljWjd1MXJ1czZ2Y0ZDdWpaSWoweWx1eGp3ejFlTU9XUmdvQ0VpClhmOGRsaVJTQVV4L204Slg5RTFYU2ZzRDQxZzZQZFZWeGlONFFORGpkZ2tzQVorVjNKVlc5QUNlWEY5ZFU3NGgKSnFSOFhKWU5Lbng2cUZtZURObXV5WXVEa245NmRlVHErS2xKT3NMb3ZEVDR6M1VNVzA5VEVwclVmdGxaSStaZApwenJZaGlVKzBiaUdTOVhVN0dyUkNzN3IrRm5KTGF1TDN3bloxUFk0SXpJaWJZOXFtUUtDQVFFQTk0Q3VjRmJxCmVTVkhJakxxdXV1eHJpT2I5TDRVZDhxdk1DcDAyZHdTRWZaY2VCcUE1NmdVa01RZ2dCQ1BYbWJOT1dRbkgyYloKUU8xOEJSWTRjbkdMVzZEL3EwYVhiQzh2WEFDVlpzZnBYcldxbjNTSGNjOWZpNnZlOGlHNGtKZEVOUmRnQlU4VwpsR2o3aDhTZ2lHSkNzbHQvTTdtZ0lqa0FCZ0xNQitsRUFrSzBWa0pJVEE0N0xzRlNScElMTnJhUkt2aG5INUVQCkVoVlViazNYZm5XSkJOTTZyci9pb1NicjgvWU1uZENZbzlBWFJOalFmU0RmMHBVYWpSNmJraXBnVVZiUDliZGYKNVhyZ2MxSlNQQlVUTkxSR3hNVEhXTlgrbGJzcWhpQysrU3RZUTN3dmFCZXBOemZ0cXhVV0lXTzNKbXVyV3BUYQpQZ1NyOTYwVVpJbjhIUUtDQVFFQSthWU1ZeSs5Ty9sWWRpb1Q0dWhFZVZaQ2pDcGdhSVpham1aUnp6SXA5eVFaCm9jam1aRk1seXJKYUlmaFNyMHFKZVZtSWtGQ0tyNDdaY0ROaGhtMFV0cWlPbGovQWtuQWowQkV1b1UwYTVxSFkKR0M2UnZVUG1xSEVTRTR5Zmpub1huczB6VnFRN1krSEhUNWR5Y2NpZzJxYUZaNk8vR3BoUUZVd01mMG41eHd3Kwo2WCtPam50eTREUTd4MmwzOGNRUmYrdkh3MGhuWllBcllqZytmR0QzQ0ZpeDZueVVtRDg1amErTURmckNOTkFBCnpZMyszV0FPQzJ1L3lNSjRydjVmazJyajBRNU5sN3NRc1Fwc3Q5WGhiTHNUTWRmeDNqQmdxbXFsN2J5Vnp3MloKM29JdFlncjVCZmtLS3lIOC9RMndMMndOeVRXQ1l6MWtyclBmZnZYUHh3S0NBUUJIMFg1TXVOdlhCWHNyc0V5dQpxci9uUVF2N0s4RHl0Y3k2RkVmT0ErNzJhVitSdGxjYllZbCtMSHNsemloY0EwYWYxYkVJaXFhV0VaT0FRbDlrCnpnL2JLYyttbXBoTDJ6Rko2QjF5TXFaRVJrRFpmazNqTjRLSkcvbFlsM0pmK3BUZk53WTA1Q3N3SzNwNWZoUDcKSDFBdFF5R1pGODhndnh1RG93SWpkWXUzZ0RXbUpodW1maWFzUFlxclVhdVJWODZ1QW1DaUoweVJPY0ZETkxGSQpUOERQdHA5N245Q2FaSm5wTThlYmI3RXJMN0hnMTIxQU1lN2d3MFZ1RjZpYTlGTDRwMUUzQXR2LzBmVVpZWlRkClBGeFRXZENEUG5wK0M3S1JManQ3cWpyZ1FMU2UrSVVsRm1DUzFsYlA0eEdGNU5KN2dwaTVjeUlWQnZRRHJhU1MKTy92OUFvSUJBUURGeXYxOWlGRlJ0eGlUWm5zakNBdFlaek9LZ2Zpb1YrcGZjRW5ZODFHMGNYR3RjTks1SWZlTApSUXVNWm9aOFEzM3dHelBMdzBSZUc3dkMzYktqSXNHS2hybVI2U2pWM090QzZwb2JTay9KOHVpWElDNXYyZUJpCkRGUGFFVXhKUWdwODB1K2Q4YmpzUmZIMzZYSFBITG4xQW9JbnZ1Q21YWTcxa0s5R0dvSS9aa0JpRjZJRzJXQUcKcXR2Qi9wbjlmdTZ1ZjB4aU9IZFRQOTBma0poUlN6SHQ5dmZmWkowR2t2RXloS2RlWEJLS2JWSjFpYzhuN2ZheQpyY2ZoYzlMU01zL2VxSTJmRU1vQk1VRGtRL0luSk5uWm44NXhhenBDWStueW0xU2pxd3EyWlh4SGdyUWFQYjlYCk1CMFNWM2R0dHU2a1krUDRTdURuWjdqaGdibk5pVXY1QW9JQkFEU0JKRUxCVkJBOS9QU0lSYXpDNmlCVm5vckoKamd6TUQxbVcrM0JYOUFFK0pOWm5QMGtqaHp2TTJ1QU1PRFRWbVJQNkF5d2ZlbGVlR0Vjc2MzSHhNKzBtQXdCdQpOY2FEbXR5aTVMbmoxSXM2VEltOWEwR0dFYWJRQjVzbFJtUU5GUFVjeFBKVGF2R0dwYWZxckxoZkJWRXYxU3lUCldiRDhyT2Yxd2QyMzZYYkl1SDhPZnU1WGtJVUR5dFdVMk8rNkN4bFJCZmFKNkQvUk1VQWk1TlZJY09ybjhxUnkKU3hNcFpBY3hBd1BpVmNVL3NSc0JqWVdPdTNleXIrcjdKZktUdnB2dTI3a1l6ODZFMlhDZUtZblpock85WDN1YgpseGN6WW9CaFpaeGRIRjdmRnoyb29UZUlxVEtKNkx4OFF2N1VHR2lhUmdIN3hrMmQ5MWdWejBGRmVKVT0KLS0tLS1FTkQgUlNBIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: "2021-02-15T04:24:32Z"
  generateName: sealed-secrets-key
  labels:
    sealedsecrets.bitnami.com/sealed-secrets-key: active
  name: sealed-secrets-keynvpcz
  namespace: kube-system
  resourceVersion: "1141"
  selfLink: /api/v1/namespaces/kube-system/secrets/sealed-secrets-keynvpcz
  uid: fd35e6a1-6cb9-4a9c-b50c-91a9a03f6cd5
type: kubernetes.io/tls
```

# Create Namespace for testing sealed-secrets

```
➜  kubectl apply -f k3s/02-namespace.yaml
namespace/foo created
```

Verify it by running 

```
➜  kubectl get ns
NAME              STATUS   AGE
default           Active   27m
kube-system       Active   27m
kube-public       Active   27m
kube-node-lease   Active   27m
foo               Active   41s
```

# Create Secret

Create a regular Secret as a template to seal it later

```
➜  echo -n 'sealed-secrets-example' | base64
c2VhbGVkLXNlY3JldHMtZXhhbXBsZQ==
```

Replace ``<token>`` with the generated toke in ``03-secret.yaml`` 

```
➜  kubectl apply -f k3s/03-secret.yaml
secret/credentials created
```

# Configure sealed-secrets

We need the key certificate to seal secrets. We can use ``kubeseal`` to fetch it from the controller at runtime or we can store it locally using the below command and use it offline.

```
➜  kubeseal --controller-namespace kube-system --fetch-cert > cert.pem
```

Decode the certificate to take a look

```yaml
➜  openssl x509 -in cert.pem -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            88:7e:e9:d9:ed:51:0a:eb:a9:20:16:43:88:1c:c0:9f
    Signature Algorithm: sha256WithRSAEncryption
        Issuer:
        Validity
            Not Before: Feb 15 04:24:32 2021 GMT
            Not After : Feb 13 04:24:32 2031 GMT
        Subject:
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:f1:5c:b3:16:4e:7a:b9:67:8d:80:3d:0e:a9:65:
                    91:93:01:a5:d2:d4:1b:0d:0c:e7:09:b8:82:04:62:
                    ff:9b:a8:a4:b4:b5:bd:5c:4d:b5:c3:0a:93:f1:c6:
                    a0:a3:df:98:ae:23:f8:cb:cb:d2:e1:8e:a4:e5:cc:
                    91:6d:0d:eb:d4:7f:61:76:6f:64:73:bd:16:84:b4:
                    72:18:a6:f0:17:31:f3:42:55:39:14:30:eb:93:18:
                    1e:f6:ad:32:f1:d8:a2:35:99:88:3a:38:8f:8f:0e:
                    1d:ba:06:b5:a4:c6:56:d6:f2:81:5e:ee:49:e8:79:
                    73:b7:9f:a7:49:8f:c5:bd:62:98:39:65:73:71:2e:
                    a7:1a:82:4f:c7:99:20:28:18:b4:24:cd:aa:bd:ad:
                    08:95:54:cd:a3:03:e2:cf:50:89:87:e2:32:29:87:
                    d0:58:bd:68:cb:f9:c2:ef:6b:1d:16:87:26:d3:62:
                    4f:d8:01:bf:bf:e2:c5:7b:38:c2:3a:00:79:78:7b:
                    c8:cd:82:b5:55:8f:70:dd:58:38:6f:de:c3:66:bb:
                    1b:21:93:43:b2:a5:9d:9f:c2:ba:95:b3:d3:7c:b2:
                    8f:bb:24:00:d6:42:75:f0:d4:c4:ce:01:e7:38:09:
                    2e:91:6d:52:e5:dd:14:f0:7a:00:ec:ce:0a:59:f6:
                    35:da:7f:2d:16:1f:70:8d:ad:d7:95:48:95:25:dc:
                    d2:7c:43:2e:34:63:59:a7:af:3d:d0:6b:e4:db:1a:
                    35:7d:2d:39:f6:16:90:b7:83:9f:59:89:bd:a6:b5:
                    be:b6:45:8c:b4:8c:e9:28:e7:83:23:51:1b:54:6a:
                    8f:e7:a7:6e:8f:d7:f0:6d:90:f3:74:db:88:69:36:
                    78:1b:2b:03:79:26:5b:8d:75:e3:fa:28:31:62:2f:
                    0b:00:b1:81:ac:fd:88:72:56:85:1a:ee:fa:3e:88:
                    c9:87:37:05:6b:16:cc:9a:18:e1:0f:48:f4:5d:8b:
                    a2:97:8f:f1:e0:c5:80:31:30:da:58:63:a6:f8:d6:
                    bb:ce:79:0d:69:69:fe:1c:f5:fe:8b:ec:d2:5a:98:
                    ac:8c:f9:af:11:ff:3b:7b:17:4e:9c:ed:a1:c9:15:
                    db:d4:e0:ce:1b:31:b4:6a:54:2e:e3:99:16:bc:cb:
                    a7:cf:46:ef:fd:8d:27:d0:d6:54:76:07:50:49:80:
                    40:06:6e:4d:78:f5:bb:db:5a:26:7d:8c:ff:af:cf:
                    94:f7:a8:ba:56:56:ab:f4:e5:14:dd:01:fb:ac:10:
                    37:26:22:64:9b:af:34:94:de:5d:b2:eb:b9:e6:b2:
                    10:24:3e:86:69:33:c3:ea:05:25:5f:ac:0e:8d:47:
                    df:6d:8b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Encipher Only
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         c4:b3:07:ca:75:bb:67:30:04:55:b9:51:9c:1f:1b:c9:d9:5c:
         08:6e:1d:75:24:e0:84:b8:74:f3:9b:cd:3a:4c:d0:38:b0:c3:
         68:2f:be:bd:35:b3:40:0e:82:5b:2c:cd:36:4a:6e:5e:9a:7d:
         af:7b:73:cb:aa:9c:16:f6:1f:5d:22:6b:4f:56:70:cb:35:b3:
         82:fd:8a:f8:18:e6:33:f3:ae:2e:d3:98:2c:d4:f8:2f:64:99:
         c4:83:d2:73:ad:25:52:7e:fb:45:5d:75:23:b7:45:63:75:0d:
         92:a4:6f:1f:b1:ae:fa:a4:12:61:91:54:d4:f6:52:4e:de:67:
         05:25:c7:4f:6f:fe:19:a5:43:f5:b7:4a:f7:f9:5f:17:ab:cd:
         0d:eb:ad:de:21:1f:63:da:62:dc:14:7c:77:71:bf:10:3c:fd:
         07:97:d2:fc:d3:d7:57:c8:33:5c:f5:1f:06:d8:77:6d:42:19:
         bf:e6:b4:e8:f3:4e:ee:dd:36:45:aa:e4:b2:43:7b:24:83:9b:
         d5:69:e2:61:48:d6:c5:d6:2a:ec:f1:91:eb:2a:e2:22:d1:af:
         2d:d4:73:ad:7f:6c:c2:d5:ab:ad:47:e1:05:26:cd:25:6e:42:
         60:c8:d9:cb:13:d0:0d:da:e1:d1:21:a6:d9:a8:80:51:34:2e:
         c7:ef:a3:4b:ad:dc:7a:18:61:8c:05:12:bc:a8:a4:94:21:20:
         8b:6b:ae:ab:95:6e:57:57:1b:ca:af:20:7f:78:c2:15:78:7d:
         77:91:23:24:4e:1e:08:c5:04:6a:68:77:37:02:1c:d9:36:e5:
         e0:2f:8a:6a:b5:4d:c6:01:6f:3c:bf:3e:36:69:dc:f1:a5:60:
         1c:84:ae:57:1c:5c:53:3d:bc:43:4f:ff:23:c5:59:ad:cb:32:
         c3:46:f8:4f:b5:f9:7b:14:51:70:49:34:6d:6f:c2:cd:7f:93:
         aa:b8:e4:dc:f7:a9:e8:52:af:40:fd:62:5d:5e:56:22:87:28:
         74:6a:0c:98:e0:61:86:28:4d:8d:93:96:ba:06:9a:42:99:38:
         d4:f0:94:a8:c9:0e:9c:ec:97:99:0f:1b:2c:8e:c4:ff:0b:c1:
         56:90:ca:8c:a4:1e:09:16:d4:02:40:e6:16:04:47:a1:3f:23:
         62:46:84:78:8e:37:1b:69:16:41:39:4f:00:23:58:ea:ca:6c:
         69:b2:83:c3:9e:f4:8f:c8:33:1d:44:e7:90:4b:14:4c:84:8c:
         ff:5f:e5:9d:3a:4c:57:79:9e:db:85:44:0c:e9:54:14:42:c1:
         a7:f9:17:c7:74:00:03:4c:4e:e4:8a:47:75:a2:0c:9e:c7:ba:
         f2:85:48:4f:d9:19:3c:17
```

# Encrypt Secret

```
➜ kubeseal < k3s/03-secret.yaml --cert cert.pem -o yaml > k3s/04-sealed-secret.yaml
```

04-sealed-secret.yaml

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: credentials
  namespace: foo
spec:
  encryptedData:
    token: AgB46venfU8xaT2KA+1FstQApxf51r9DnGF33ZXkewaeaPg6KckWCEtaZ6sfcILPbpGZRmMW29R5lWF0HLzbnWB3ZmhouUjibWqEfeskVpCmsKntXNHI0h//8sLwoECgqreDaU34WjMGJzMIjdWZXGbym57OqJqwDGTBxBQJG2lwrRQ1EjS57juhnYuNm0V7HPEPDKCfUUhhwfqZ+GMrAJVK1JWPzztQEJY0RVeUw1AzL6PTK8HBHMSSl680ZNwC+IAZUBM36vIxHbnehbm9yB4QAeceqMMQDp8tPaK5Qw+440hYm2OdfX9+Y5ePNmXyN1h6XWMmUWUToneZk/5yTn9o9hnDelznmGl3DslAi4lzCTew56eagikXQGZE9IDpoYv1ptKTMNYjdESYSdynMTHjZYiNM5dXpCRwWxXwmuMQU3NLlmEaOupUPeNSavewoU6NvVm1Cq+DBv7SSSMQUjvHgBSFNDLRpBh3egvOqp2RKXyUq/1OCByHwlhg/HW0ZpzATUHrTDa4aRDgnaEwo+vazrOtPKlSHSop57daF7+Xx54bxguXzHQAgCBk6prqEQSuMRNOgCknuxNnFaNSYBgdsmrxTyreNwCXhAYzGja6LGSibXP843ZS3Ph1YfDtFGIzbXqB8bxd5Pbx1N0tJCIGrqIs1MoB94hdmCYWH+qe8tFZ2VgrLjOPWzQzn9EH+++ZQWLSnKQyzZyyRR4gArDnXXt3cCn3
  template:
    metadata:
      creationTimestamp: null
      name: credentials
      namespace: foo
    type: Opaque
```

```
➜  kubectl apply -f k3s/04-sealed-secret.yaml
sealedsecret.bitnami.com/credentials created
```

If you update credentials in sealedsecrets, it will also update that in secrets.
```
➜  kubectl get sealedsecrets -n foo
NAME          AGE
credentials   40s
```

```
➜  kubectl get secrets -n foo
NAME                  TYPE                                  DATA   AGE
default-token-4g77w   kubernetes.io/service-account-token   3      35m
credentials           Opaque                                1      27m
```

```
➜  kubectl get secrets credentials -o yaml -n foo
```

```yaml
apiVersion: v1
data:
  token: c2VhbGVkLXNlY3JldHMtZXhhbXBsZQ==
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"token":"c2VhbGVkLXNlY3JldHMtZXhhbXBsZQ=="},"kind":"Secret","metadata":{"annotations":{},"name":"credentials","namespace":"foo"},"type":"Opaque"}
  creationTimestamp: "2021-02-15T04:43:23Z"
  name: credentials
  namespace: foo
  resourceVersion: "1925"
  selfLink: /api/v1/namespaces/foo/secrets/credentials
  uid: b80b9465-d35b-422e-8ab4-6cd73d3be5b3
type: Opaque
```

To decode the token

```
➜  echo "c2VhbGVkLXNlY3JldHMtZXhhbXBsZQ==" | base64 -d
sealed-secrets-example
```

# Create an Express application

server.js

```js
'use strict';

const express = require('express');
const PORT = 8080;
const HOST = '0.0.0.0';
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});
app.listen(PORT, HOST);

console.log(`Running on http://${HOST}:${PORT}`);
console.log(`The Token is ${process.env.TOKEN}`);
```

Dockerfile

```dockerfile
FROM node:14

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```

.dockerignore

```
node_modules
npm-debug.log
```

Build the image

```
➜ docker build -t wingkwong/sealed-secrets-example .
```

Your image will be listed by Docker
```
➜ docker images
REPOSITORY                                                                 TAG                 IMAGE ID            CREATED             SIZE
wingkwong/sealed-secrets-example                                           latest              a7710ac9878f        2 minutes ago       946MB
```

Run the image

```
➜ docker run -p 8080:8080 -d wingkwong/sealed-secrets-example
1b59a182529ab9b6022eae20b1c3cc9f8c97f25e133334158f3cdf2b97d6046f
```

```
➜ docker ps
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                          NAMES
1b59a182529a        wingkwong/sealed-secrets-example   "docker-entrypoint.s…"   24 seconds ago      Up 20 seconds       0.0.0.0:8080->8080/tcp                         unruffled_boyd
```

``<token>`` is undefined as we don't define it in environment.

```
➜ docker logs 1b59a182529a
Running on http://0.0.0.0:8080
The Token is undefined
```

Push to Docker Hub
```
➜ docker push wingkwong/sealed-secrets-example
The push refers to repository [docker.io/wingkwong/sealed-secrets-example]
9c504e9b67e2: Pushed
9443efda2621: Pushed
4f5d94c4d9a0: Pushed
c190d83c7139: Pushed
7ad435f34cd1: Mounted from library/node
c52fdd5ebc39: Mounted from library/node
5faa7f35f547: Mounted from library/node
9b88fe065b35: Mounted from library/node
4ca605ea46de: Mounted from library/node
601f04850201: Mounted from library/node
846bd2f3b216: Mounted from library/node
2b3e667f5e92: Mounted from library/node
e891be0c59b2: Mounted from library/node
latest: digest: sha256:0af214e064458380d4a5ea740176c7312e9bcf638b8aaa207b401bb9e64ea35c size: 3047
```

# Deploy an Express application to test sealed-secrets

05-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sealed-secrets-example-deployment
  namespace: foo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sealed-secrets-example
  template:
    metadata:
      labels: # labels to select/identify the deployment
        app: sealed-secrets-example
    spec:     # pod spec                  
      containers: 
      - name: sealed-secrets-example
        image: wingkwong/sealed-secrets-example:latest # image we pushed
        ports:
        - containerPort: 8080
        env:
        - name: TOKEN
          valueFrom:
            secretKeyRef:
              key: token
              name: credentials
```


```
➜ kubectl apply -f k3s/05-deployment.yaml
deployment.apps/sealed-secrets-example-deployment created
```

```
➜ kubectl get pods -n foo
NAME                                                  READY   STATUS    RESTARTS   AGE
sealed-secrets-example-deployment-6bb786499c-2rbtq   1/1     Running   0          9m24s
```

```
➜ kubectl describe pods sealed-secrets-example-deployment-6bb786499c-2rbtq -n foo
Normal   Pulling    6m30s                  kubelet, k3d-sealed-secrets-example-server  Pulling image "wingkwong/sealed-secrets-example:latest"
Normal   Pulled     43s                    kubelet, k3d-sealed-secrets-example-server  Successfully pulled image "wingkwong/sealed-secrets-example:lates
```

```
➜ kubectl logs -l app=sealed-secrets-example -n foo
Running on http://0.0.0.0:8080
The Token is sealed-secrets-example
```

# Clean up
```
k3d delete -n sealed-secrets-example
```

# References

- ["Sealed Secrets" for Kubernetes](https://github.com/bitnami-labs/sealed-secrets)