---
layout: post
category : 리눅스
tagline: "리눅스 프로그래밍"
tags : [리눅스, clang, sublimetext, llvm]
---
{% include JB/setup %}

##시작하기전에..
일단 [지훈현서 블로그에 있는 "가벼운 개발환경 만들기"](http://mcchae.egloos.com/10991779)를 참고해서 기본적인 리눅스 개발 환경을 만들어둡니다.. 이 단계는 지금 생략 하고.. 필요하면 차후 별도 글로 올리도록 하겠습니다. 추가로 설명 하자면... 지훈현서님의 블로그는 VM기준으로 최적화된 우분투 환경입니다.

저는 최근에 clang에 관심을 갖게 되어서 clang을 설치합니다. 설치 방법의 출처는 [여기](http://solarianprogrammer.com/2013/01/17/building-clang-libcpp-ubuntu-linux/) 내용을 번역 한것입니다.

우선 설치에 필요한 패키지를 가져옵니다.
    
	sudo apt-get install g++ subversion cmake
    

##Clang과 llvm설치하기
그 후 Clang 과 llvm 최신 버전을 받습니다.
    
    cd ~
    mkdir Clang && cd Clang
    svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
    cd llvm/tools
    svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
    
그리고 빌드...
    
    cd ..
    cd ..
    mkdir build && cd build
    ../llvm/configure --prefix=/usr/clang_3_3 --enable-optimized --enable-targets=host
    make -j 8
    
시간이 꽤 오래 걸립니다...
빌드가 끝났으면 Clang을 설치 해줍니다.
    
	sudo make install
    
시스템에 clang을 추가해줍니다..
    
	export PATH=/usr/clang_3_3/bin:$PATH
    
물론.. .bashrc 파일 마지막에 추가 해줘도 됩니다..

이제 libc++을 설치해봅니다.
g++정보 가져오는 방법입니다.
    
	echo | g++ -Wp,-v -x c++ - -fsyntax-only
    
그러면 아래와 같이 출력됩니다.
    
    ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
    ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../../x86_64-linux-gnu/include"
    #include "..." search starts here:
    #include <...> search starts here:
     /usr/include/c++/4.6
     /usr/include/c++/4.6/x86_64-linux-gnu/.
     /usr/include/c++/4.6/backward
     /usr/lib/gcc/x86_64-linux-gnu/4.6/include
     /usr/local/include
     /usr/lib/gcc/x86_64-linux-gnu/4.6/include-fixed
     /usr/include/x86_64-linux-gnu
     /usr/include
    End of search list.
    
여기서 **/usr/include/c++/4.6**가 Include폴더가 되고 **/usr/include/c++/4.6/x86_64-linux-gnu/.**은 64비트일 경우 include경로가 됩니다.

##libc++설치하기
아래와 같이 실행해줍니다
    
    cd ~/Clang
    svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx
    mkdir build_libcxx && cd build_libcxx
    
아래부분의 include를 위에서 찾은 값으로 변경해주세요..
    
    CC=clang CXX=clang++ cmake -G "Unix Makefiles" -DLIBCXX_CXX_ABI=libsupc++ -DLIBCXX_LIBSUPCXX_INCLUDE_PATHS="/usr/include/c++/4.6/;/usr/include/c++/4.6/x86_64-linux-gnu/" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr $HOME/Clang/libcxx
    
마무리...
    
    make -j 8
    sudo make install
    

##Clang테스트
clang으로 컴파일 하는 방법은 아래와 같습니다.
    
	clang++ -std=c++11 -stdlib=libc++ <your_program_name>
    
간단히 테스트 해보도록 하자.c++11에서만 지원하는 lamda기능을 이용해보도록 합니다.
    
    #include <iostream>
    
    using namespace std;
    
    int main()
    {
        cout << [](int m, int n) { return m + n;} (2,4) << endl;
        return 0;
    }
    
이걸 컴파일 하면 다음과 같습니다.
    
    clang++ -std=c++11 -stdlib=libc++ example_001.cpp -o example_001
    ./example_001
    6
    
굳이 clang을 쓸 필요는 없습니다.. 다만.. clang은 계속 gcc보다 한발짝 먼저 표준을 구현해주는 것 같습니다.. 해당 포스트 작성할 당시에는 gcc는 아래 코드를 컴파일 못했습다..
    
    #include <iostream>
    #include <regex>
    #include <string>
    
    using namespace std;
    
    int main()
    {
        string input;
        regex rr("((\\+|-)?[[:digit:]]+)(\\.(([[:digit:]]+)?))?((e|E)((\\+|-)?)[[:digit:]]+)?");
        //As long as the input is correct ask for another number
        while(true)
        {
            cout<<"Give me a real number!"<<endl;
            cin>>input;
            //Exit when the user inputs q
            if(input=="q")
                break;
            if(regex_match(input,rr))
                cout<<"float"<<endl;
            else
            {
                cout<<"Invalid input"<<endl;
            }
        }
    }
    
자.. 이제.. g++대신 clang을 사용해보도록합니다.
    
	sudo update-alternatives --install /usr/bin/g++ c++ /usr/clang_3_2/bin/clang++ 90
    
이렇게 설정해두면 g++불러도 clang++ 이 동작합니다.

##기타 설정
개인적으로 vim에 익숙하지 않고.. 배우기도 싫은 사람에겐 [sublimetext](http://www.sublimetext.com/)를 권합니다. 기본적으로 유료 라이센스가 존재하지만 무료로 사용해도 큰 문제는 없습니다.
    
    sudo add-apt-repository ppa:webupd8team/sublime-text-3
    sudo apt-get update
    sudo apt-get install sublime-text-installer
    
설치후 유용한 plugin들이 많습니다..
ctrl+\`를 누른 후 
    
	import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
    
를 입력해주면 plugin관리자가 설치가 됩니다.. 자세한 사항은 [여기](https://sublime.wbond.net/)를 참고 하면 됩니다.

##마무리
이상.. 리눅스 기본적인 c++개발 환경입니다. 