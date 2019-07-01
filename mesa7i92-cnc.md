# CNC Controller with Mesa7I92 and Gecko G540
```This post describes connecting bCNC to a Gecko G540 4 axis controller with a Mesa7i92 ethernet to db25```

# Connections
To avoid issues with buffering we decided to use ethernet and UDP to talk between our CNC controller and the motors.

# Configure Routing
The Mesa7i92 board has a default IP address of 192.168.1.121 which will probably conflict with your private IP space. 

Add an appropriate route to 192.168.1.121.

'auto eno1
iface eno1 inet static
	address 192.168.1.1/24
	netmask 255.255.255.0
	gateway 192.168.1.1
'

Then disable your wifi or router when attempting to connect to the Mesa7i92 board.

Check the route table to ensure nothing conflicts or is missing.

'route'

Will return
'
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     *               255.255.255.0   U     0      0        0 eno1
'

# Laptop to Mesa7i92 via Ethernet
* Ping 192.168.1.121 and watch the 4 leds on the Mesa board increment using binary math. The ethernet plug orange light will remain fixed and the green plug light will blink for each ping. 

'ping -c 30 192.168.1.121'

Ping will Tx to the Mesa7i92 and the board with Rx back. You can see this when running 'ifconfig' and you'll see the Tx and Rx volumes have increased. This confirms you can talk to the board and it can reply.

Note Telnet won't work as it's tring to TCP and the Mesa7i92 board only supports UDP.

* The Mesa7i92 board only supports ARP, Ping (ICMP) and UDP so you can't use Telnet or other TCP connection methods to check that you have comms. Use Netcat to check for the transmission and receipt of UDP packets. Note the Mesa7i92 communicates using UDP on port 27181 and IP 192.168.1.121.

'nc -vnzu 192.168.1.121 27181'

Will return 'Connection to 192.168.1.121 27181 port [udp/*] succeeded!'

* Now that you have a connection and can send and receive UDP packets it's time to check the Mesa7i92 configuration. I was interested in the Gecko G540 and the Mesa7i92 supports two Gecko G540 controllers for a total of 4 plus 4 axes. You'll need to install the Mesaflash utility from www.mesanet.com.

'mesaflash --device 7i92 --addr 192.168.1.121 --readhmid'

Will return the following verbose configuration. There is nothing to indicate specifically what configuration your board is flashed with. Apart from checking the pin assignments.

'
Configuration Name: HOSTMOT2

General configuration information:

  BoardName : MESA7I92
  FPGA Size: 9 KGates
  FPGA Pins: 144
  Number of IO Ports: 2
  Width of one I/O port: 17
  Clock Low frequency: 100.0000 MHz
  Clock High frequency: 200.0000 MHz
  IDROM Type: 3
  Instance Stride 0: 4
  Instance Stride 1: 64
  Register Stride 0: 256
  Register Stride 1: 256

Modules in configuration:

  Module: DPLL
  There are 1 of DPLL in configuration
  Version: 0
  Registers: 7
  BaseAddress: 7000
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: WatchDog
  There are 1 of WatchDog in configuration
  Version: 0
  Registers: 3
  BaseAddress: 0C00
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: IOPort
  There are 2 of IOPort in configuration
  Version: 0
  Registers: 5
  BaseAddress: 1000
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: QCount
  There are 2 of QCount in configuration
  Version: 2
  Registers: 5
  BaseAddress: 3000
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: StepGen
  There are 10 of StepGen in configuration
  Version: 2
  Registers: 10
  BaseAddress: 2000
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: PWM
  There are 2 of PWM in configuration
  Version: 0
  Registers: 5
  BaseAddress: 4100
  ClockFrequency: 200.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

  Module: LED
  There are 1 of LED in configuration
  Version: 0
  Registers: 1
  BaseAddress: 0200
  ClockFrequency: 100.000 MHz
  Register Stride: 256 bytes
  Instance Stride: 4 bytes

Configuration pin-out:

IO Connections for P2
Pin#  I/O   Pri. func    Sec. func       Chan      Pin func        Pin Dir

 1      0   IOPort       None           
14      1   IOPort       PWM              0        PWM             (Out)
 2      2   IOPort       StepGen          0        Step/Table1     (Out)
15      3   IOPort       None           
 3      4   IOPort       StepGen          0        Dir/Table2      (Out)
16      5   IOPort       StepGen          4        Step/Table1     (Out)
 4      6   IOPort       StepGen          1        Step/Table1     (Out)
17      7   IOPort       None           
 5      8   IOPort       StepGen          1        Dir/Table2      (Out)
 6      9   IOPort       StepGen          2        Step/Table1     (Out)
 7     10   IOPort       StepGen          2        Dir/Table2      (Out)
 8     11   IOPort       StepGen          3        Step/Table1     (Out)
 9     12   IOPort       StepGen          3        Dir/Table2      (Out)
10     13   IOPort       QCount           0        Quad-A          (In)
11     14   IOPort       QCount           0        Quad-B          (In)
12     15   IOPort       QCount           0        Quad-IDX        (In)
13     16   IOPort       None           

IO Connections for P1
Pin#  I/O   Pri. func    Sec. func       Chan      Pin func        Pin Dir

 1     17   IOPort       None           
14     18   IOPort       PWM              1        PWM             (Out)
 2     19   IOPort       StepGen          5        Step/Table1     (Out)
15     20   IOPort       None           
 3     21   IOPort       StepGen          5        Dir/Table2      (Out)
16     22   IOPort       StepGen          9        Step/Table1     (Out)
 4     23   IOPort       StepGen          6        Step/Table1     (Out)
17     24   IOPort       None           
 5     25   IOPort       StepGen          6        Dir/Table2      (Out)
 6     26   IOPort       StepGen          7        Step/Table1     (Out)
 7     27   IOPort       StepGen          7        Dir/Table2      (Out)
 8     28   IOPort       StepGen          8        Step/Table1     (Out)
 9     29   IOPort       StepGen          8        Dir/Table2      (Out)
10     30   IOPort       QCount           1        Quad-A          (In)
11     31   IOPort       QCount           1        Quad-B          (In)
12     32   IOPort       QCount           1        Quad-IDX        (In)
13     33   IOPort       None           
'

# Now configure the serial port
* Had to use MAKEDEV to create /dev/ttyS0 which was missing on my Ubuntu 16 machine.

* Let's start tcpdump while we configure our serial to ethernet mapping

'tcpdump -i any -vnn port 27181' We can test this with 'nc -vnzu 192.168.1.121 27181' or ping -c 4 192.168.1.121

or

'tcpdump -nvvv -i any -c 3 host 10.0.3.1' and alternatives at https://bencane.com/2014/10/13/quick-and-practical-reference-for-tcpdump/

* Create a virtual serial port
`sudo socat -d -d -d pty,link=/dev/ttyS0,raw udp:192.168.1.121:27181`

which will produce:

2019/04/18 21:59:26 socat[2411] I socat by Gerhard Rieger - see www.dest-unreach.org
2019/04/18 21:59:26 socat[2411] I This product includes software developed by the OpenSSL Project for use in the OpenSSL Toolkit. (http://www.openssl.org/)
2019/04/18 21:59:26 socat[2411] I This product includes software written by Tim Hudson (tjh@cryptsoft.com)
2019/04/18 21:59:26 socat[2411] I setting option "symbolic-link" to "/dev/ttyS0"
2019/04/18 21:59:26 socat[2411] I setting option "raw"
2019/04/18 21:59:26 socat[2411] I openpty({5}, {6}, {"/dev/pts/19"},,) -> 0
2019/04/18 21:59:26 socat[2411] N PTY is /dev/pts/19
2019/04/18 21:59:26 socat[2411] N opening connection to AF=2 192.168.1.121:27181
2019/04/18 21:59:26 socat[2411] I starting connect loop
2019/04/18 21:59:26 socat[2411] I socket(2, 2, 17) -> 7
2019/04/18 21:59:26 socat[2411] N successfully connected from local address AF=2 192.168.1.1:55048
2019/04/18 21:59:26 socat[2411] I resolved and opened all sock addresses
2019/04/18 21:59:26 socat[2411] N starting data transfer loop with FDs [5,5] and [7,7]





* Let's try COM1 or /dev/ttyS0

'setserial -a /dev/ttyS0'

will return

setserial -a /dev/ttyS0




# Mesa7i92 to Gecko G540 with Parallel Port (DB25)


