## uaf - 8 pt ##

Flaga: `flag`

Kod źródłowy **uaf.c**:

```cpp
#include <fcntl.h>
#include <iostream> 
#include <cstring>
#include <cstdlib>
#include <unistd.h>
using namespace std;

class Human{
private:
	virtual void give_shell(){
		system("/bin/sh");
	}
protected:
	int age;
	string name;
public:
	virtual void introduce(){
		cout << "My name is " << name << endl;
		cout << "I am " << age << " years old" << endl;
	}
};

class Man: public Human{
public:
	Man(string name, int age){
		this->name = name;
		this->age = age;
        }
        virtual void introduce(){
		Human::introduce();
                cout << "I am a nice guy!" << endl;
        }
};

class Woman: public Human{
public:
        Woman(string name, int age){
                this->name = name;
                this->age = age;
        }
        virtual void introduce(){
                Human::introduce();
                cout << "I am a cute girl!" << endl;
        }
};

int main(int argc, char* argv[]){
	Human* m = new Man("Jack", 25);
	Human* w = new Woman("Jill", 21);

	size_t len;
	char* data;
	unsigned int op;
	while(1){
		cout << "1. use\n2. after\n3. free\n";
		cin >> op;

		switch(op){
			case 1:
				m->introduce();
				w->introduce();
				break;
			case 2:
				len = atoi(argv[1]);
				data = new char[len];
				read(open(argv[2], O_RDONLY), data, len);
				cout << "your data is allocated" << endl;
				break;
			case 3:
				delete m;
				delete w;
				break;
			default:
				break;
		}
	}

	return 0;	
}
```
* Output z checksec:
```
uaf@ubuntu:~$ checksec uaf
[*] '/home/uaf/uaf'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE
```
* Program na początku alokuje pamięć na (tworzy) dwa obiekty: `man` i `woman`
* Warto zwrócić uwagę na rozmiar alokacji pamięci dla obiektów, który w obydwu przypadkach wynosi `0x18 (24)` - przyda się to w następnych krokach

```asm
   0x0000000000400ef7 <+51>:	lea    r12,[rbp-0x50]
   0x0000000000400efb <+55>:	mov    edi,0x18
   0x0000000000400f00 <+60>:	call   0x400d90 <_Znwm@plt>
```
