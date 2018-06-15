# -Security-Engineering-Password-Cracker
 - 2018년 1학기 성균관대학교 김형식교수님의 보안공학 수업, 1번째 과제
 - PBKDF2 알고리즘과 HMAC-SHA1 해쉬 함수를 이용하여 암호화한 패스워드를 해킹하는 과제
 - 2018 first semester Sungkyunkwan University Professor Hyoungshick Kim's Security Engineering class, 1st assignment
 - Hacking the encrypted password using PBKDF2 algorithm and HMAC-SHA1 hash function

## 1. Problem
 - This is a small programming assignment to learn why choosing a good password selection policy is important.
 
 - Password is most often used for user authentication. The use of password is easy to implement and does not require extra hardware support.
 - You will write a password cracker that does a brute force or guessing attack on a set of passwords randomly selected from real password datasets.
 - In the tested dataset, a password is at least 6 characters long, contains one or more numerical digits.
 - Your program must be written in C/C++/Python and work on a password file with k users where k <= 100000. The format of the password file is (with one entry per line):
   
   ![image](https://user-images.githubusercontent.com/26705935/41457050-5b9fc0fe-70bd-11e8-8114-aa73035a8d2a.png)
     
 - That is, given the input file (\hashedPasswords.txt") consisting of the hash values of real users' passwords, your program should create the output file (\Passwords.txt") consisting of real users' passwords (see the following example).
 
   ![image](https://user-images.githubusercontent.com/26705935/41457096-812a6d10-70bd-11e8-86ee-bd1de2b92d01.png)

 - I will use the PBKDF2 algorithm (see [this](http://www.ietf.org/rfc/rfc2898.txt)) for protecting passwords in the input file (\hashedPasswords.txt") with 500 iterations and 16 byte keys; HMAC-SHA1 and the string seclab were used as its underlying PRF and the salt, respectively.
 
 - You must make sure the submitted file size is less than or equal to 2MB.
 - You will be judged by the number of correct answers in the output file (\Passwords.txt") created by your program. There is the k=100 seconds time limit for running the code (we will measure the system time).
   
## 2. Environment
 - language : C++
 - IDE : Microsoft Visual studio 2017
 
## 3. Result
### (1) 과제 접근
#### 1. 알고리즘을 이용하여 역으로 계산함으로써 원래의 암호를 얻어낸다.
 - 문제에 주어진 것과 같이 PBKDF2 알고리즘과 SHA1 해시 함수의 구조를 이해하였고, 주어진 해시 값을 이용하여 이 구조를 차례대로 역으로 수행함으로써 원래의 암호를 얻어낼 수 있을 것이라 생각하였다. 이는 기존 암호 알고리즘은 풀기 힘들지만, assignment1은 salt 값과 iteration 값을 알고 있기 때문에 풀 수 있을 것이라 예상하였다. 
 - 하지만 이 값들을 알고 있다 하더라도 해시 함수 자체가 단일 방향성을 가지고 있기 때문에, 해시 값을 이용하여 본래의 암호를 계산해낼 수 없다는 결론을 얻었다.
 
#### 2. Brute force Attack
 - Brute force 공격은 조합 가능한 모든 패스워드를 대입해보는 것으로 원래의 암호를 얻어내는 공격 기법이다. 임의의 단어를 PBKDF2 알고리즘에 넣었을 때 나오는 해시 값을, 문제에서 주어진 해시 값과 비교하여 같은 경우에 본래의 암호를 얻어낼 수 있다. 
 - 기본적으로 모든 해시는 brute force 공격으로 뚫을 수 있다. 하지만 수행 시간이 매우 오래 걸린다는 단점이 있다.
 
#### 3. Rainbow Attack
 - Brute force 공격은 각 임의의 단어를 알고리즘에 넣고, 결과로 얻어낸 해시 값을 주어진 해시 값과 비교하는 것이라면, rainbow table을 이용한 공격은 미리 가능한 암호 조합과 그 해시 값을 다 계산한 테이블을 이용하여 비교만 수행하도록 한다. 이 공격은 사전에 가능한 암호 조합을 저장해놓은 rainbow table(dictionary)에 따라 수행 능력이 달라지는데, 단순히 본래의 암호가 rainbow table에 있으면 얻어낼 수 있고, 없으면 얻어낼 수 없다.[1]
 - 어떤 유형의 암호든 뚫을 수 있는 Brute force 공격에 비해 성공 확률이 떨어지지만, 시간을 엄청나게 단축할 수 있다. 따라서 이 rainbow table을 이용한 공격을 통해 본래의 암호를 얻어내는 방법을 이용할 것이다.
 
### (2) 수행 단계
#### 1. Rainbow table 구축
 - 가능한 암호의 리스트로는 가장 널리 쓰이는 것으로 선정된 12000여개의 암호들 및 다른 암호 리스트들(mostpasswords.txt)을 이용하였다.[2] Assignment1에 명시되어 있는 대로, 본래의 암호는 6글자 이상의 영문, 숫자, 특수기호의 조합으로, 한 개 이상의 숫자가 포함되어 있는 형태이다. [2]의 암호 리스트들 중에는 이러한 조건을 만족하지 않는 암호들이 많기 때문에 이를 맞추고, 또한 약간의 변형(특수기호가 없는 경우, 특수 기호 !을 맨 뒤에 붙이는 등)을 통해 새로이 암호 리스트(passwordlist.txt)를 저장하였다. 저장한 암호 리스트 내의 암호의 총 개수는 44401개로, 그림1과 같다.
 
 - 또한 PBKDF2 알고리즘을 수행하는 라이브러리[3]를 이용하여 암호 리스트에 존재하는 특정 암호에 따른 해시 값을 계산하고, 이를 저장하였다.(hashlist.txt) 저장된 해시 암호의 개수는 암호 리스트 내의 암호의 개수와 동일한 44401개로, 그림2와 같다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41457415-83bdd0ac-70be-11e8-8cc1-d37a6f0bb359.png)
 
 - 이러한 과정은 GenerateData.c 소스코드를 통해 구현할 수 있다. 소스 내에서 PBKDF2 알고리즘을 적용하기 위해서 fastpbkdf2.c 및 fastpbkdf2.h 라는 외부 참조 라이브러리를 이용하였다. 이 두 파일은 모두 GenerateData.c 소스코드가 해당되어 있는 프로젝트 폴더 내부에 위치해야 한다.

#### 2. Assignment1 input 파일 읽기
 - Input 파일(hashedPasswords.txt)에는 번호, 이름 그리고 해시 암호 정보가 있다. 그 형식은 아래와 같이 되어있다. 이러한 형식을 이용하여 user의 이름과 해시 암호를 matrix에 저장하였다.

     **[number]:[username]:[hashed password]**

 - 1)에서 저장한 해시 암호 리스트(hashlist.txt)를 search하면서 문제에서 주어진 해시 암호와 같은 것이 있는지를 알아보고, 같은 것이 있다면 그에 해당하는 암호를 암호 리스트(passwordlist.txt)에서 불러오고, 이름과 matching하여 output 파일(Passwords.txt)에 출력한다.

 - 만일 해시 암호 리스트에 주어진 해시 암호와 동일한 항목이 없다면, search는 실패한다. 이는 예상되는 암호 리스트(rainbow table)가 잘못되었다는 것을 알려준다. 하지만 구현에서는 사용자의 이름에 숫자 1과 특수 기호 ‘!’을 첨가한 문자열을 주어진 해시 암호에 대한 암호라고 guessing하여 이를 출력한다. 이는 많은 사용자가 자신의 이름을 암호에 넣는다는 특성을 이용한 것이다.

 - 출력된 파일은 그림 3과 같다. 출력 형식은 Input 파일 형식과 동일하고, hashed password 대신 예측한 암호를 출력하도록 하였다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41457468-b99ec06e-70be-11e8-9930-815a56d33ff0.png)
   
 - 또한 사용자가 k명(k<=100,000)인 경우 프로그램 실행 시간이 k/100초 이하여야 한다는 과제 조건에 따라, time.h 라이브러리의 함수를 이용하여 총 실행 시간을 체크하였고, 이를 출력하였다. 출력한 결과는 그림 4와 같이 나타났다. 10000명의 사용자에 대해 프로그램 실행에 대략 0.81~0.84초 정도가 소요됨으로써 문제의 조건을 만족하였다. 또한 새로운 프로젝트 상에서 실행한 결과 약 2초정도가 소요됨으로써 문제의 조건을 만족하였다.
 
   ![image](https://user-images.githubusercontent.com/26705935/41457511-d89cce84-70be-11e8-96da-2d5beb7b2272.png)
   
 ### (3) 소스 코드 및 기능
 #### 1. GenerateData.c
 - GenerateData.c는 rainbow table을 구축하는 소스코드로, 실제 테스트할 때 구동할 필요가 없다. 보고서에 명시 및 제출하는 것은 단순히 rainbow table을 구축한 방식을 설명하기 위함이다.
 - SHA1 해시 함수를 이용하는 PBKDF2 알고리즘을 적용하기 위해 "fastpbkdf2.h" 및 "fastpbkdf2.c"라는 오픈소스 라이브러리[3]를 include하였다.

#### 2. TestHash.c
 - TestHash.c는 input 파일(hashedPasswords.txt)을 읽어오고, 주어진 해시 암호 값을 이미 저장된 rainbow table(passwordlist.txt와 hashilst.txt)내의 해시 암호들과 단순 비교함으로써 얻은 본래의 암호를 output 파일인 Passwords.txt에 출력 및 저장하는 코드이다. 
  - 실제 성능을 테스트할 때 이 코드를 구동하면 된다. 단, 코드가 위치하는 프로젝트 폴더 내에 "passwordlist.txt" 및 “hashlist.txt"가 같이 위치해야 한다.
 
## 4. Conclusion
 - 예시 값들을 넣고 이를 실행한 결과, 3쪽의 그림3과 같이 해시 암호가 table내에 있는 경우에는 잘 찾음으로써 본래의 암호를 성공적으로 출력하였다. 하지만 만일 해시 값에 해당하는 본래의 암호가 rainbow table에 없는 경우엔 본래의 암호를 찾는데에 실패한다. 프로그램을 통해 만든 Rainbow table (passwordlist.txt 및 hashlist.txt)이 의미있는 암호들을 포함하고 있고, 암호 해독에 있어서 효과적으로 작용할 것이다.
 
## 5. References
 [1] http://www.codeok.net/%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%B4%EC%95%88%EC%9D%98%20%EA%B8%B0%EC%88%A0
 
 [2] https://github.com/danielmiessler/SecLists/tree/master/Passwords
 
 [3] https://github.com/ctz/fastpbkdf2
