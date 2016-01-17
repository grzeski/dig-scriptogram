slow_elb.pl find slow responses times from AWS ELB logs. 

First of all get some ELB access logs, for example from Publishing ELB and Warden ELB from their S3 buckets for 28th August:

```
$ mkdir pub-elb
$ aws s3 sync s3://pub-ext-elb/AWSLogs/719728721003/elasticloadbalancing/eu-west-1/2015/08/28 pub-elb
$ mkdir warden-elb
$ aws s3 sync s3://warden-az1/AWSLogs/719728721003/elasticloadbalancing/eu-west-1/2015/08/28 warden-elb
```

Then run slow_elb.pl on all the log files you've downloaded to show processing time above 5 secs:

```
$ perl slow_elb.pl -t 5 *.log | tee warden-28th
```

This will output something like below. It will only print the value of the 3 processing time values if it is greater than the threshold. 

```
========719728721003_elasticloadbalancing_eu-west-1_publishing-external-elb-https_20150828T0005Z_54.154.130.111_4i6lq586.log=========
2015-08-27T23:57:16.890169Z - 54.633297 -
2015-08-27T23:58:09.425847Z - 2.188004 -
2015-08-27T23:57:17.066736Z - 54.609933 -
2015-08-27T23:58:00.416854Z - 11.293701 -
2015-08-27T23:57:16.857100Z - 54.880973 -
2015-08-27T23:58:09.539551Z - 2.255389 -
2015-08-27T23:57:52.561908Z - 19.233133 -
2015-08-27T23:57:56.632728Z - 15.213939 -
```

If you just want to isolate slow tie processing within the ELB internals pass the -1 flag. If you just want to isolate backend delays pass -2. 

If you want to step through the results chronologically (since AWS emits many access logs files) and remove the headers sort it:

```
$ cat warden-28th | grep -v ========= | sort -n > warden-28th-sorted
```

You can pass the following options to slow_elb.pl: 

```
 -1              Compare time of elb internal processing time (sec). Defaults if none specified. 
 -2              Compare time of backend processing time (sec). Defaults if none specified. 
 -3              Compare time of response processing time (sec). Defaults if none specified. 
 -threshold n    n in seconds of time threshold to compare against. Default is 1 sec. 
 -short          Display time and 3 responses times only in results. Default. 
 -verbose        Display whole log statement in results.
```