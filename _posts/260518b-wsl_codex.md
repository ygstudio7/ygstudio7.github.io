참고: https://www.youtube.com/watch?v=KzE62JwPtc4

https://www.youtube.com/watch?v=iLWKWoD0oHU

https://github.com/develckm86/kosmo_l04_spring_smuboard_2025_06

npm: js 라이브러리를 제공하는 패키지 매니저

codex는 node.js로 만들어짐

wsl에서 node js를 설치해야 함

L3: nvm  설치

L6: version 22 이상 설치

L9: codex 설치

```bash
# https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl
# Install Node.js in WSL (via nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash

# In a new tab or after exiting and running `wsl` again to install Node.js
nvm install 22

# Install and run Codex in WSL
npm i -g @openai/codex
codex
```



nvm 설치후에는 nvm을 쓸수있도록 아래와 같이 설정하라고 나온다

$export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

$nvm

$nvm install 22

이제 npm 명령을 쓸수 있게 되었다

다음과 같이 npm을 update하라고 하면 진행한다

``` bash
npm install -g npm@11.14.1
```



$ codex 실행하면, 내 chatgpt 요금제와 연결된다.



```bash
$git clone https://github.com/develckm86/kosmo_l04_spring_smuboard_2025_06.git
```



java를 다운로드한다

```bash
$java -version
: java 확인 X
$sudo apt-get update
$apt list openjdk-21*
$sudo apt-get install openjdk-21-jdk
$java -version

```

