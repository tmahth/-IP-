**컨테이너 네트워크 이해를 위해 IP 명령어를 이용한 수동 구성**

## 1. veth(virtual ethernet)

veth는 리눅스의 버츄얼 이더넷 인터페이스를 의미합니다. veth는 쌍으로 만들어지며 네트워크 네임스페이스들을 터널로서 연결하거나, 물리 디바이스와 다른 네트워크 네임스페이스의 장비를 연결하는 용도로 사용할 수 있습니다.

리눅스에서는 ip를 사용해 veth 타입의 가상 네트워크 장비를 생성할 수 있습니다. veth는 쌍으로 만들어지므로, \<veth0_name\>와 \<veth1_name\>을 적절한 이름으로 치환해줍니다.

예) # ip link add \<veth0_name\> type veth peer name \<veth1_name\>


## 2. 구성
### 2.1 네트워크 네임스페이스
veth라는 버츄얼 이더넷를 만든다
<pre><code># ip link add veth0 type veth peer name veth1
# ip link
veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000 link/ether 72:d1:e3:44:82:60 brd ff:ff:ff:ff:ff:ff
veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000 link/ether c6:62:6f:65:3c:c5 brd ff:ff:ff:ff:ff:ff
</code></pre>

vnet0라는 네트워크 네임스페이스를 만든다
<pre><code># ip netns add vnet0
# ip netns
vnet0
</code></pre>

veth1 디바이스를 네임스페이스 vnet0로 이동한다.
<pre><code># ip link set veth1 netns vnet0
# ip link // veth1 디바이스가 안보이고, veth0 상태는 down이다

veth0@ … _ **state DOWN** _ …

**# ip netns exec vnet0 ip link // veth1 이 보이고, veth1 상태는 down이다

veth0 … _ **state DOWN** _ …

**# ip link set veth0 up //veth0을 up 시키면 lowerlayerdown 상태가 된다

**# ip link

veth0 …docke _ **state LOWERLAYERDOWN** _ …

**# ip netns exec vnet0 ip link //veth1은 down이다

veth1@if451: \&lt;BROADCAST,MULTICAST\&gt; mtu 1500 qdisc noop _ **state DOWN** _ mode DEFAULT group default qlen 1000

link/ether 16:db:42:bf:bd:ee brd ff:ff:ff:ff:ff:ff link-netnsid 0

**# ip netns exec vnet0 ip link set veth1 up

**# ip link

veth0 … _ **state UP** _ …

**# ip netns exec vnet0 ip link

veth1 … _ **state UP** _ ...

**# ip addr add 10.1.1.1/24 dev veth0 //veth0 디바이스에 ip부여

**# ip a

veth0 … inet 10.1.1.1/24

**# ip netns exec vnet0 ip addr add 10.1.1.2/24 dev veth1

**# ip netns exec vnet0 ip a

veth1 … inet 10.1.1.2/24 …

**# ip netns exec vnet0 ping 8.8.8.8 // google 로 핑이 나가지 않음

**# ip netns exec vnet0 ip route add default via 10.1.1.1 //vnet0 netns에 gw설정

**# iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j MASQUERADE //masq설정

**# ip netns exec vnet0 ping 8.8.8.8 // google로 핑이 잘 나감

**# ip link add br0 type bridge** // 이름이 br0인 브릿지 디바이스를 만든다.

**# ip link set br0 up**

**# ip netns add vnet0**

**# ip link add veth0 type veth peer name veth1**

**# ip link set veth1 netns vent0**

**# ip netns exec vnet0 ip a add 10.1.1.2/24 dev veth1**

**# ip netns exec vnet0 ip link set dev lo up**

**# ip netns exec vnet0 ip link set dev veth1 up**

**# ip link set veth0 master br0**

**# ip link set veth0 up**

**# ip netns add vnet1**

**# ip link add veth2 type veth peer name veth3**

**# ip link set veth3 netns vent1**

**# ip netns exec vnet1 ip a add 10.1.1.3/24 dev veth3**

**# ip netns exec vnet1 ip link set dev lo up**

**# ip netns exec vnet1 ip link set dev veth3 up**

**# ip link set veth2 master br0**

**# ip link set veth2 up**

**# ip a add 10.1.1.1/24 brd 10.1.1.255 dev br0**

**# ip netns exec vent0 ip route add default via 10.1.1.1**

**# ip netns exec vent1 ip route add default via 10.1.1.1**

**# sysctl -w net.ipv4.ip\_forward=1**

**# iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -j MASQUERADE**

참조

[https://www.44bits.io/ko/keyword/veth](https://www.44bits.io/ko/keyword/veth)

[https://www.44bits.io/ko/post/container-network-1-uts-namespace](https://www.44bits.io/ko/post/container-network-1-uts-namespace)

[https://www.44bits.io/ko/post/container-network-2-ip-command-and-network-namespace](https://www.44bits.io/ko/post/container-network-2-ip-command-and-network-namespace)
