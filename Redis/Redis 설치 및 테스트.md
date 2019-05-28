Redis 설치하기
 version=5.0.5
 bits=64
리눅스에서 설치하기 (CentOS7)
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make
make install

make 실행시 다음과 같은 오류가 발생한다면
In file included from adlist.c:34:0:
zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: 그런 파일이나 디렉터리가 없습니다


 yum -y install epel-release 
 yum -y install varnish 
실행 후 make install를 돌려봅니다.
그래도 오류가 난다면 
cd deps
  make hiredis jemalloc linenoise lua
cd .. 
다음과 같이 입력 후 다시 make install 을 하면 오류 없이 설치됩니다.


make test시 다음과 같은 오류가 발생한다면 
You need tcl 8.5 or newer in order to run the Redis test

아래 명령어로 tcl을 설치해줍니다.
yum install -y tcl

참고로 tcl은:
The Tcl package contains the Tool Command Language, a robust general-purpose scripting language.


실행하기
아래 명령어로 손쉽게 redis 서버를 실행시킬 수 있습니다.
redis-server

설정변경하기 
redis-stable 디렉토리 안에 redis.conf 파일에서 변경 가능합니다.
port 

bind
defualt로 localhost에서만 접근 가능하다. 
bind를 주석처리하면 모든 ip에서 접근이 가능하다.

데이터 추가하기 
src/redis-server를 실행한다

set [key] [value]
get [key]
와 같이 사용하면 된다.
incr [key]
del [key]

시간 설정 
set resource:lock "Redis Demo"
expire resource:lock [시간 초단위] 
위와 같이 진행하면 120 초 이후에 
삭제됨
ttl resource:lock를 실행해보면 
해당 키 값에 걸려있는 만료 시간을 확인할 수 있다.
기본적으로는 남은 시간이 리턴되며,
이미 삭제되었을 경우 -2, 만료 시간이 세팅되어 있지 않을 경우 -1을 리턴한다.

[list 데이터 구조]
llen [key] : 리스트의 길이를 리턴한다.
lpush [key] : 리스트의 첫번째에 값을 추가한다. 
rpush [key] : 리스트의 마지막에 값을 추가한다. 
lpop [key] : 첫 번째 요소를 반환하고 삭제처리한다.
rpop [key] : 마지막 요소를 반환하고 삭제 처리한다.

LRANGE [key] [index1] [index2]
index1 - index2까지의 value를 반환한다. 
index2에 -1을 추가할 경우, 리스트의 끝까지 반환한다.

[regular set 데이터 구조]
list와 유사하나, 순서가 없고 요소의 중복을 허용하지 않느다는 특징이 있다.
sadd [key] [value] : 데이터 추가
srem [key] [value] : 데이터 삭제
sismember [key] [value] : 데이터가 set에 존재하는지 리턴 1(존재)/0(존재하지 않음)
smembers [key] : 모든 데이터를 반환.
sunion [key] [key] : 두 개 이상의 set을 합치고 요소를 반환한다.

[sorted set 데이터 구조]
set은 굉장히 편리한 데이터 타입이지만, 정렬이 되지 않는다는 문제가 있습니다. 그래서 redis 1.2부터는 sorted sets이 소개되었습니다.
sorted set은 regular set과는 비슷하지만 각 요소가 score를 가지고 있다는 점이 특징입니다. 이 score는 요소들을 정렬할 때 사용됩니다. 
zadd [key] [score] [value] : 데이터 추가
zrange [key] [index1] [index2] : score 기준으로 정렬하여 index1 - index2까지를 반환

[hash 데이터 구조]
hset [key] [string field] [string value]
hgetall [key]
hmset [key] [string field1] [string value1] [string field2] [string value2] ...
HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
hget [key] [string field] : 요소 리턴
해시 필드의 숫자값은 문자열과 동일하게 취급되며, 이 값을 원자 단위로 증가시키는 연산이 있음.


keys * 쌓여있는 모든 데이터 확인
flushall 전체 삭제
dbsize 용량 
