Dangling Calico WorkloadEndpoints present some WorkloadEndpoints are not allocated to ERU workload. We could look for them via a [scripts/check_calico.py](https://github.com/projecteru2/core/blob/master/scripts/check_calico.py) and reap them manually.

```bash
$ ./check_calico.py -h
usage: check_calico.py [-h] [-e ERU_ETCD_ENDPOINTS] -p ERU_ETCD_PREFIX

optional arguments:
  -h, --help            show this help message and exit
  -e ERU_ETCD_ENDPOINTS, --eru-etcd-endpoints ERU_ETCD_ENDPOINTS
                        the ERU ETCD endpoints
  -p ERU_ETCD_PREFIX, --eru-etcd-prefix ERU_ETCD_PREFIX
                        the ERU ETCD root prefix
```

