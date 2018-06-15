1. Password Cracker

hashlist.txt
passwordlist.txt
TestHash.c

상위 3개의 파일을 한 프로젝트 폴더 내에 위치시키고, hashedPasswords.txt가 있는 상태에서 TestHash.c를 실행.

VC 17 환경에서 프로젝트 속성은 Release - x64로 구현 및 실행.


----------------------------------------------------------------------------------------------------------

2. References

fastpbkdf2.c
fastpbkdf2.h
GenerateData.c

상위 3개의 파일은 주요 기능을 하는 code는 아님. 각 코드의 기능은 ReadMe.md에 기재되어 있음.

rainbow table을 만드는 과정에 대해서는, OpenSSL을 설치하고 프로젝트의 포함 및 참조 라이브러리를 설정하는 작업이 필요함.
