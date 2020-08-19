# Day20project


// API(어플리케이션 프로그래밍 인터페이스)
// 조작을 하기위한 통로?


게임만들기

#pragma once

#ifndef stdafx_h
#define stdafx_h

#include<iostream>
#include<string>
#include<stdlib.h>
#include<time.h>
#include<stdio.h>
#include<vector>
#include<Windows.h>
#include<assert.h>
#include<stack>//STL스택
#include<queue>

#include"Keyboard.h"

using namespace std;

// SAFE_DELETE(포인터)
#define SAFE_DELETE(p){if(p){delete(p); (p) = nullptr ;}}//p가 0이 아니면 if문에 들어간다.


#endif


#pragma once
class Table
{
public:
	static void Create();
	static void Delete();
	static Table* Get();

	void PrintText(int x, int y, const char* str);
	void DrawTable();

	void DrawCharacter(class Character* character);

	int GetRight() { return right; }
	int GetFloor() { return floor; }

private:
	Table();
	~Table();


	
	static Table* instance;

	string top;
	string middle;
	string bottom;
	int right = 0;
	int floor = 0;

};




#include "stdafx.h"
#include "Table.h"
#include "Character.h"

Table* Table::instance = nullptr;

void Table::Create()
{
	if(instance == nullptr)
		instance = new Table();
}

void Table::Delete()
{
	SAFE_DELETE(instance);
}

Table * Table::Get()
{
	return instance;
}

void Table::PrintText(int x, int y, const char * str)
{
	COORD coord = { x * 2, y }; 
	SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), coord);
	cout << str << endl;
}

void Table::DrawTable()
{
	int index = 1;
	PrintText(0, 0, top.c_str());

	for (index; index < floor; index++)
		PrintText(0, index, middle.c_str());


	PrintText(0, index, bottom.c_str());

}
//┏┓┗┛━┃
void Table::DrawCharacter(Character * character)
{
	PrintText(character->GetPosition().X, character->GetPosition().Y, character->GetSymbol());
}

Table::Table()
{
	CONSOLE_CURSOR_INFO cursor = { 1, false };
	SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &cursor);

	floor = 28;
	right = 100;
	//////////////////////////
	top = "┏";
	for (int i = 0; i < right; i++)
	{
		top += "━";
	}
	top += "┓";
	///////////////////////////
	middle = "┃";
	for (int i = 0; i < right; i++)
	{
		middle += " ";
	}
	middle += "┃";
	////////////////////////////

	bottom += "┗";
	for (int i = 0; i < right; i++)
	{
		bottom += "━";
	}
	bottom += "┛";
	////////////////////////////
}

Table::~Table()
{
}




#pragma once

#define MAX_INPUT_KEY 255
#define MAX_INPUT_MOUSE 8 

class Keyboard
{
public:
	static Keyboard* Get(); 

	static void Create(); 
	static void Delete(); 

	void Update(); 

	bool Down(DWORD key) { return keyMap[key] == KEY_INPUT_STATUS_DOWN; } 
	bool Up(DWORD key) { return keyMap[key] == KEY_INPUT_STATUS_UP; } 
	bool Press(DWORD key) { return keyMap[key] == KEY_INPUT_STATUS_PRESS; } 

private:
	Keyboard();
	~Keyboard();

	static Keyboard* instance; 

	byte keyState[MAX_INPUT_KEY];
	byte keyOldState[MAX_INPUT_KEY];
	byte keyMap[MAX_INPUT_KEY];

	enum
	{
		KEY_INPUT_STATUS_NONE = 0,
		KEY_INPUT_STATUS_DOWN,
		KEY_INPUT_STATUS_UP,
		KEY_INPUT_STATUS_PRESS,
	};
};




#include "stdafx.h"
#include "Keyboard.h"

Keyboard* Keyboard::instance = NULL;

Keyboard * Keyboard::Get()
{
	return instance;
}

void Keyboard::Create()
{
	instance = new Keyboard();
}

void Keyboard::Delete()
{
	SAFE_DELETE(instance);
}

void Keyboard::Update()//키보드의 상태를 보는 함수
{
	memcpy(keyOldState, keyState, sizeof(keyOldState));
	ZeroMemory(keyState, sizeof(keyState));
	ZeroMemory(keyMap, sizeof(keyMap));


	GetKeyboardState(keyState);

	for (DWORD i = 0; i < MAX_INPUT_KEY; i++) 
	{
		//이 키가 단순히 눌렸는지 안눌렸는
		byte key = keyState[i] & 0x80;//마지막 비트값이 1로 세팅이 됬느냐? 물어보는 16진수(10진수로 128)(2진수로 1000 0000)
		keyState[i] = key ? 1 : 0;

		int oldState = keyOldState[i];
		int state = keyState[i];

		if (oldState == 0 && state == 1) 
			keyMap[i] = KEY_INPUT_STATUS_DOWN;
		else if (oldState == 1 && state == 0)
			keyMap[i] = KEY_INPUT_STATUS_UP;
		else if (oldState == 1 && state == 1)
			keyMap[i] = KEY_INPUT_STATUS_PRESS;
		else
			keyMap[i] = KEY_INPUT_STATUS_NONE;
	}
	GetKeyState(0);
}

Keyboard::Keyboard()
{
	ZeroMemory(keyState, sizeof(keyState));
	ZeroMemory(keyOldState, sizeof(keyOldState));
	ZeroMemory(keyMap, sizeof(keyMap));//현재 키보드의 상태와 전의 상태를 비교하여 map에다가 넣어 주는 것이다.
}

Keyboard::~Keyboard()
{

}



#pragma once
class Character
{
public:

	Character(const char* symbol, COORD position,bool bPlayer = false);
	~Character();

	const char* GetSymbol() { return symbol; }//심볼을 어느 위치에 찍을 것인가?
	COORD GetPosition() { return position; }

	bool Move();//누른대로 가게
	void AutoMove();//랜덤을 돌려서 어느 한 방향에 이동할 수 있도록.
	void DropThePoop();
	void SetRightAndFloor(int right, int floor)
	{
		this->right = right;
		this->floor = floor;
	}


	void CheckWall();//벽과 좌표가 같으면 하나씩 밀어내기.
private:
	const char* symbol;//캐릭터를 문자로 표현(그래픽이 없어서)
	COORD position;//캐릭터의 위치 좌표를 position에 담아두겠다.
	int right = 0;
	int floor = 0;
	int speed = 1;
	bool bPlayer = false;

};







#include "stdafx.h"
#include "Character.h"


Character::Character(const char * symbol, COORD position, bool bPlayer)
	:symbol(symbol), position(position),bPlayer(bPlayer)
{

}

Character::~Character()
{
}

bool Character::Move()
{
	if (Keyboard::Get()->Down('W'))
	{
		position.Y--;
	

		return true;
	}
	else if (Keyboard::Get()->Down('S'))
	{
		position.Y++;
		return true;
	}

	if (Keyboard::Get()->Down('A'))
	{
		position.X--;
		return true;
	}
	else if (Keyboard::Get()->Down('D'))
	{
		position.X++;
		return true;
	}

	return false;
}

void Character::AutoMove()
{
	int dir = rand() % 4;

	switch (dir) //42, 11
	{
		case 0: //위로
		{
			position.Y--;
			break;
		}
		case 1: //아래로
		{
			position.Y++;
			break;
		}
		case 2: //왼쪽
		{
			position.X--;
			break;
		}
		case 3: //오른쪽
		{
			position.X++;
			break;
		}
	}
	CheckWall();
}

void Character::DropThePoop()
{
	position.Y += speed;
	CheckWall();//원 밖으로 안나가게 제한을 검
}

void Character::CheckWall()
{
	if (position.X <= 0)
		position.X += speed;

	if (position.X >= (right / 2) + 1)
		position.X -= speed;

	if (position.Y <= 0)
		position.Y += speed;

	if (position.Y >= floor && bPlayer == true )
		position.Y -= speed;
	else if (position.Y >= floor && bPlayer == false)
	{
		position.Y = rand() % 3 + 1;
		position.X = rand() % (right / 2);
	}
}









#include"stdafx.h"
#include"Keyboard.h"
#include"Table.h"
#include"Character.h"

//☺☻★☆
//인텔리센스(.하면 나오는 것 ,색,등등..)

int main()
{

	srand((unsigned int)time(NULL));

	Keyboard::Create();//keyborad인스턴스 생성
	Table::Create();


	Character* player = new Character("★", { 5,5 },true);//구조체이기 때문에 가능
	//Character* enemy = new Character("☆", { 5,7 });//구조체이기 때문에 가능

	vector<Character*>poops;
	for (int i = 0; i < 30; i++)//좌표를 랜덤하게
	{
		int rndX = rand() % (Table::Get()->GetRight() / 2);
		int rndY = rand() % 3 + 1;

		poops.push_back(new Character("☆", { (SHORT)rndX,(SHORT)rndY }));
		poops.back()->SetRightAndFloor(Table::Get()->GetRight(), Table::Get()->GetFloor());
		

	}
	player->SetRightAndFloor(Table::Get()->GetRight(), Table::Get()->GetFloor());

	Table::Get()->DrawTable();

	Table::Get()->DrawCharacter(player);
	//Table::Get()->DrawCharacter(enemy);
	for (int i = 0; i < poops.size(); i++)
	{
		Table::Get()->DrawCharacter(poops[i]);
	}

	while (true)
	{
		Sleep(60);

		Keyboard::Get()->Update();

		for (int i = 0; i < poops.size(); i++)
		{
			poops[i]->DropThePoop();
		}


		if (player->Move())
			player->CheckWall();
			//enemy->AutoMove();

		Table::Get()->DrawTable();
		Table::Get()->DrawCharacter(player);
			//Table::Get()->DrawCharacter(enemy);
		for (int i = 0; i < poops.size(); i++)
		{
			Table::Get()->DrawCharacter(poops[i]);
		}


		
	}

	Keyboard::Delete();

	Table::Delete();


	return 0;
}

#숙제

Queue..
스킬 입력용 큐를 구현하라
- 캐릭터 액션이 끝날때 다음 스킬을 쓸 수 있도록하는 스킬 큐를 만드시오
- 총 5개 까지 쌓임 이후는 무시됨.

스킬을 저장하는 queue문제..
스킬은 print로 대충 만들자










