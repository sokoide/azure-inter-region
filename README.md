# Azure Inter Region Quick Performance Test

## Environment

* sokoide-use2
  * http server which returns X KB random string (configurable) in US East2
  * The http server is fronted by Traefik to add TLS/mTLS layer using <https://github.com/sokoide/tls-overhead>
* sokoide-use2b
  * client in US East2
* sokoide-usec
  * another client in US Central

## Test Date

* 2025/3/30


## Test Result Summary

* 10x latency getting from USC to USE2 when the data size is small
* 2x total throughput pushing from USC when the data size is big

### PUSH

#### iPerf3 push

| Test          | From  | Sender    | Receiver |
|---------------|-------|-----------|----------|
| iperf3        | use2b | 933 MB/s  | 929 MB/s |
| iperf3        | usec  | 556 MB/s  | 552 MB/s |

#### gRPC push

| Test          | From  | Time   |
|---------------|-------|--------|
| 4KB x 1000    | use2b | 360ms  |
| 4KB x 1000    | usc   | 900ms  |
| 4KB x 2000    | use2b | 730ms  |
| 4KB x 2000    | usc   | 1300ms |
| 4KB x 4000    | use2b | 1520ms |
| 4KB x 4000    | usc   | 2300ms |
| 1MB x 100     | use2b | 9.1s   |
| 1MB x 100     | usc   | 10.3   |
| 1MB x 200     | use2b | 18.1s  |
| 1MB x 200     | usc   | 23.7s  |
| 1MB x 400     | use2b | 36.7s  |
| 1MB x 400     | usc   | 54.4s  |

### HTTP GET over plain HTTP, HTTPS, mTLS

| Test              | From  | Received          | Sent          | reqs/s |
|-------------------|-------|-------------------|---------------|--------|
| 4KB plain HTTP    | use2b | 103 MB/s          | 2.7 MB/s      | 24354  |
| 4KB plain HTTP    | usc   | 12 MB/s           | 307 KB/s      | 2762   |
| 4KB HTTPS         | use2b | 74 MB/s           | 1.5 MB/s      | 17672  |
| 4KB HTTPS         | usec  | 12 MB/s           | 237 KB/s      | 2759   |
| 4KB mTLS          | use2b | 74 MB/s           | 1.5 MB/s      | 17723  |
| 4KB mTLS          | usc   | 11 MB/s           | 245 KB/s      | 2711   |


## Test Results Raw Data

### PUSH

#### iperf3 push

```bash
# on the use2 server
$ iperf3 -s -p 50051

# on use2b client
$ iperf3 -c sokoide-use2.eastus2.cloudapp.azure.com -p 50051
Connecting to host sokoide-use2.eastus2.cloudapp.azure.com, port 50051
[  5] local 10.0.0.5 port 46658 connected to 68.154.120.178 port 50051
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   118 MBytes   991 Mbits/sec    0   1.09 MBytes
[  5]   1.00-2.00   sec   110 MBytes   926 Mbits/sec    0   1.15 MBytes
[  5]   2.00-3.00   sec   110 MBytes   919 Mbits/sec    0   1.15 MBytes
[  5]   3.00-4.00   sec   111 MBytes   931 Mbits/sec    0   1.15 MBytes
[  5]   4.00-5.00   sec   110 MBytes   924 Mbits/sec    0   1.21 MBytes
[  5]   5.00-6.00   sec   110 MBytes   926 Mbits/sec    0   1.27 MBytes
[  5]   6.00-7.00   sec   109 MBytes   917 Mbits/sec    0   1.27 MBytes
[  5]   7.00-8.00   sec   111 MBytes   930 Mbits/sec    0   1.27 MBytes
[  5]   8.00-9.00   sec   110 MBytes   923 Mbits/sec    0   1.27 MBytes
[  5]   9.00-10.00  sec   111 MBytes   932 Mbits/sec    0   1.34 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.09 GBytes   933 Mbits/sec    0             sender
[  5]   0.00-10.00  sec  1.08 GBytes   929 Mbits/sec                  receiver

iperf Done.

# on usc client
$ iperf3 -c sokoide-use2.eastus2.cloudapp.azure.com -p 50051
Connecting to host sokoide-use2.eastus2.cloudapp.azure.com, port 50051
[  5] local 10.0.0.4 port 43662 connected to 68.154.120.178 port 50051
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  50.5 MBytes   423 Mbits/sec    0   8.05 MBytes
[  5]   1.00-2.00   sec  65.2 MBytes   547 Mbits/sec    0   8.05 MBytes
[  5]   2.00-3.00   sec  70.5 MBytes   591 Mbits/sec    0   8.05 MBytes
[  5]   3.00-4.00   sec  70.0 MBytes   587 Mbits/sec    0   8.05 MBytes
[  5]   4.00-5.00   sec  67.0 MBytes   562 Mbits/sec    0   8.05 MBytes
[  5]   5.00-6.00   sec  67.6 MBytes   567 Mbits/sec    0   8.05 MBytes
[  5]   6.00-7.00   sec  67.6 MBytes   567 Mbits/sec    0   8.05 MBytes
[  5]   7.00-8.00   sec  66.4 MBytes   557 Mbits/sec    0   8.05 MBytes
[  5]   8.00-9.00   sec  68.6 MBytes   576 Mbits/sec    0   8.05 MBytes
[  5]   9.00-10.00  sec  69.0 MBytes   579 Mbits/sec    0   8.05 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   662 MBytes   556 Mbits/sec    0             sender
[  5]   0.00-10.04  sec   661 MBytes   552 Mbits/sec                  receiver

iperf Done.
```

#### gRPC push

* used <https://github.com/sokoide/grpc> -> `Push`

```bash
# server on use2
$ ./server
2025/03/30 03:42:27 server listening at [::]:50051

# client on use2b or usc
$ ./client -addr sokoide-use2.eastus2.cloudapp.azure.com:50051 -gos 1000
2025/03/30 03:36:32 Greeting: Hello defaultName
2025/03/30 03:36:33 Push: 351 ms

$ ./client -addr sokoide-use2.eastus2.cloudapp.azure.com:50051 -gos 100 -pushlen 1048576 -deadline 1m
2025/03/30 04:19:41 deadline: 1m
2025/03/30 04:19:41 Greeting: Hello defaultName
2025/03/30 04:19:50 Push: 9054 ms


```

### HTTP GET over plain HTTP, HTTPS, mTLS

* Test result when sokoide-use2 server return 4KB string per GET request

#### No TLS

* from use2b

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s script.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)


     data_received..................: 1.0 GB 103 MB/s
     data_sent......................: 27 MB  2.7 MB/s
     http_req_blocked...............: avg=6.58µs   min=622ns    med=953ns    max=16.44ms p(90)=1.52µs  p(95)=1.94µs
     http_req_connecting............: avg=2.71µs   min=0s       med=0s       max=12.16ms p(90)=0s      p(95)=0s
     http_req_duration..............: avg=4.05ms   min=349.31µs med=3.55ms   max=35.13ms p(90)=7.02ms  p(95)=8.88ms
       { expected_response:true }...: avg=4.05ms   min=349.31µs med=3.55ms   max=35.13ms p(90)=7.02ms  p(95)=8.88ms
     http_req_failed................: 0.00%  0 out of 243631
     http_req_receiving.............: avg=822.74µs min=10.65µs  med=529.71µs max=18.45ms p(90)=1.8ms   p(95)=2.54ms
     http_req_sending...............: avg=10.65µs  min=2.93µs   med=4.4µs    max=11.66ms p(90)=10.07µs p(95)=21.75µs
     http_req_tls_handshaking.......: avg=0s       min=0s       med=0s       max=0s      p(90)=0s      p(95)=0s
     http_req_waiting...............: avg=3.22ms   min=281.31µs med=2.71ms   max=32.76ms p(90)=5.8ms   p(95)=7.52ms
     http_reqs......................: 243631 24354.579205/s
     iteration_duration.............: avg=4.09ms   min=390.28µs med=3.58ms   max=47.41ms p(90)=7.07ms  p(95)=8.93ms
     iterations.....................: 243631 24354.579205/s
     vus............................: 100    min=100         max=100
     vus_max........................: 100    min=100         max=100


running (10.0s), 000/100 VUs, 243631 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```

* from usc

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s script.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)


     data_received..................: 117 MB 12 MB/s
     data_sent......................: 3.1 MB 307 kB/s
     http_req_blocked...............: avg=162.39µs min=727ns   med=1.27µs  max=62.16ms  p(90)=1.81µs   p(95)=2.22µs
     http_req_connecting............: avg=133.95µs min=0s      med=0s      max=49.94ms  p(90)=0s       p(95)=0s
     http_req_duration..............: avg=35.89ms  min=26.42ms med=35.65ms max=67.17ms  p(90)=43.12ms  p(95)=45.37ms
       { expected_response:true }...: avg=35.89ms  min=26.42ms med=35.65ms max=67.17ms  p(90)=43.12ms  p(95)=45.37ms
     http_req_failed................: 0.00%  0 out of 27734
     http_req_receiving.............: avg=158.86µs min=14.14µs med=70.86µs max=10.43ms  p(90)=300.59µs p(95)=475.79µs
     http_req_sending...............: avg=7.66µs   min=3.94µs  med=6.33µs  max=571.99µs p(90)=9.46µs   p(95)=13.07µs
     http_req_tls_handshaking.......: avg=0s       min=0s      med=0s      max=0s       p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=35.72ms  min=26.32ms med=35.53ms max=67.14ms  p(90)=42.94ms  p(95)=45.24ms
     http_reqs......................: 27734  2762.142799/s
     iteration_duration.............: avg=36.09ms  min=26.47ms med=35.69ms max=111.4ms  p(90)=43.29ms  p(95)=45.46ms
     iterations.....................: 27734  2762.142799/s
     vus............................: 100    min=100        max=100
     vus_max........................: 100    min=100        max=100


running (10.0s), 000/100 VUs, 27734 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```

#### One Way TLS (HTTPS)

* from use2b

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s --insecure-skip-tls-verify script_tls.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script_tls.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)


     data_received..................: 741 MB 74 MB/s
     data_sent......................: 15 MB  1.5 MB/s
     http_req_blocked...............: avg=21.11µs min=124ns    med=241ns   max=67.46ms p(90)=321ns   p(95)=347ns
     http_req_connecting............: avg=1.62µs  min=0s       med=0s      max=8.98ms  p(90)=0s      p(95)=0s
     http_req_duration..............: avg=5.59ms  min=508.42µs med=5.12ms  max=62.07ms p(90)=9.23ms  p(95)=10.97ms
       { expected_response:true }...: avg=5.59ms  min=508.42µs med=5.12ms  max=62.07ms p(90)=9.23ms  p(95)=10.97ms
     http_req_failed................: 0.00%  0 out of 176803
     http_req_receiving.............: avg=1.8ms   min=16.7µs   med=1.44ms  max=24.78ms p(90)=3.49ms  p(95)=4.53ms
     http_req_sending...............: avg=17.38µs min=6.58µs   med=13.47µs max=9.41ms  p(90)=23.93µs p(95)=29.78µs
     http_req_tls_handshaking.......: avg=14.65µs min=0s       med=0s      max=51.97ms p(90)=0s      p(95)=0s
     http_req_waiting...............: avg=3.77ms  min=0s       med=3.35ms  max=60.38ms p(90)=6.42ms  p(95)=7.71ms
     http_reqs......................: 176803 17671.725181/s
     iteration_duration.............: avg=5.64ms  min=547.5µs  med=5.16ms  max=82.15ms p(90)=9.27ms  p(95)=11.02ms
     iterations.....................: 176803 17671.725181/s
     vus............................: 100    min=100         max=100
     vus_max........................: 100    min=100         max=100


running (10.0s), 000/100 VUs, 176803 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```

* from usc

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s --insecure-skip-tls-verify script_tls.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script_tls.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)


     data_received..................: 116 MB 12 MB/s
     data_sent......................: 2.4 MB 237 kB/s
     http_req_blocked...............: avg=345.02µs min=192ns   med=336ns   max=131.96ms p(90)=444ns    p(95)=490ns
     http_req_connecting............: avg=136.86µs min=0s      med=0s      max=58.51ms  p(90)=0s       p(95)=0s
     http_req_duration..............: avg=35.72ms  min=26.65ms med=32.93ms max=63.15ms  p(90)=46.13ms  p(95)=55.09ms
       { expected_response:true }...: avg=35.72ms  min=26.65ms med=32.93ms max=63.15ms  p(90)=46.13ms  p(95)=55.09ms
     http_req_failed................: 0.00%  0 out of 27726
     http_req_receiving.............: avg=160.07µs min=20.97µs med=107.1µs max=6.72ms   p(90)=278.07µs p(95)=396.44µs
     http_req_sending...............: avg=16.14µs  min=7.62µs  med=14.17µs max=946.84µs p(90)=22.92µs  p(95)=29.93µs
     http_req_tls_handshaking.......: avg=173.27µs min=0s      med=0s      max=68.08ms  p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=35.55ms  min=26.55ms med=32.77ms max=62.99ms  p(90)=45.92ms  p(95)=54.96ms
     http_reqs......................: 27726  2759.437532/s
     iteration_duration.............: avg=36.11ms  min=26.7ms  med=32.97ms max=190.65ms p(90)=46.54ms  p(95)=55.22ms
     iterations.....................: 27726  2759.437532/s
     vus............................: 100    min=100        max=100
     vus_max........................: 100    min=100        max=100


running (10.0s), 000/100 VUs, 27726 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```

#### mTLS

* from use2b

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s --insecure-skip-tls-verify script_mtls.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script_mtls.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)

WARN[0000] tlsAuth.domains option could be removed in the next releases, it's recommended to leave it empty and let k6 automatically detect from the provided certificate. It follows the Go's NameToCertificate deprecation - https://pkg.go.dev/crypto/tls@go1.17#Config.

     data_received..................: 744 MB 74 MB/s
     data_sent......................: 15 MB  1.5 MB/s
     http_req_blocked...............: avg=33.35µs min=125ns    med=241ns   max=99.88ms p(90)=323ns   p(95)=349ns
     http_req_connecting............: avg=3.31µs  min=0s       med=0s      max=9.62ms  p(90)=0s      p(95)=0s
     http_req_duration..............: avg=5.56ms  min=430.41µs med=5.09ms  max=36.85ms p(90)=9.17ms  p(95)=10.86ms
       { expected_response:true }...: avg=5.56ms  min=430.41µs med=5.09ms  max=36.85ms p(90)=9.17ms  p(95)=10.86ms
     http_req_failed................: 0.00%  0 out of 177298
     http_req_receiving.............: avg=1.78ms  min=17.1µs   med=1.4ms   max=23.28ms p(90)=3.5ms   p(95)=4.49ms
     http_req_sending...............: avg=16.96µs min=6.61µs   med=13.32µs max=10.02ms p(90)=23.76µs p(95)=29.61µs
     http_req_tls_handshaking.......: avg=24.74µs min=0s       med=0s      max=81.94ms p(90)=0s      p(95)=0s
     http_req_waiting...............: avg=3.76ms  min=0s       med=3.37ms  max=34.77ms p(90)=6.37ms  p(95)=7.61ms
     http_reqs......................: 177298 17723.897077/s
     iteration_duration.............: avg=5.63ms  min=456.64µs med=5.12ms  max=106.2ms p(90)=9.22ms  p(95)=10.92ms
     iterations.....................: 177298 17723.897077/s
     vus............................: 100    min=100         max=100
     vus_max........................: 100    min=100         max=100


running (10.0s), 000/100 VUs, 177298 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```

* from usc

```bash
$ ~/repo/k6/k6 run -u 100 -d 10s --insecure-skip-tls-verify script_mtls.js

         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: script_mtls.js
        output: -

     scenarios: (100.00%) 1 scenario, 100 max VUs, 40s max duration (incl. graceful stop):
              * default: 100 looping VUs for 10s (gracefulStop: 30s)

WARN[0000] tlsAuth.domains option could be removed in the next releases, it's recommended to leave it empty and let k6 automatically detect from the provided certificate. It follows the Go's NameToCertificate deprecation - https://pkg.go.dev/crypto/tls@go1.17#Config.

     data_received..................: 114 MB 11 MB/s
     data_sent......................: 2.5 MB 245 kB/s
     http_req_blocked...............: avg=358.99µs min=168ns   med=331ns    max=134.83ms p(90)=453ns    p(95)=536ns
     http_req_connecting............: avg=138.48µs min=0s      med=0s       max=49.29ms  p(90)=0s       p(95)=0s
     http_req_duration..............: avg=36.35ms  min=26.75ms med=35.75ms  max=66.02ms  p(90)=45.51ms  p(95)=46.53ms
       { expected_response:true }...: avg=36.35ms  min=26.75ms med=35.75ms  max=66.02ms  p(90)=45.51ms  p(95)=46.53ms
     http_req_failed................: 0.00%  0 out of 27238
     http_req_receiving.............: avg=162.47µs min=20.72µs med=105.76µs max=9.03ms   p(90)=289.03µs p(95)=405.89µs
     http_req_sending...............: avg=16.6µs   min=7.39µs  med=13.86µs  max=543.38µs p(90)=25.06µs  p(95)=33.75µs
     http_req_tls_handshaking.......: avg=194.65µs min=0s      med=0s       max=92.5ms   p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=36.18ms  min=26.62ms med=35.62ms  max=65.91ms  p(90)=45.35ms  p(95)=46.26ms
     http_reqs......................: 27238  2710.502851/s
     iteration_duration.............: avg=36.76ms  min=26.81ms med=35.8ms   max=176.1ms  p(90)=45.57ms  p(95)=46.88ms
     iterations.....................: 27238  2710.502851/s
     vus............................: 100    min=100        max=100
     vus_max........................: 100    min=100        max=100


running (10.0s), 000/100 VUs, 27238 complete and 0 interrupted iterations
default ✓ [======================================] 100 VUs  10s
```
