# Target: Fruit

Target: Fruit - змейка на с++

Как играть:
1. Клонировать репозиторий
3. Скомпилировать
4. Запустить

**Команда для компиляции: g++ -o targetFruit targetFruit.cpp -lncurses**

____

#include <iostream>
#include <ncurses.h>

using namespace std;

bool gameOver; // флаг завершения игры
bool gameExit; // флаг выхода из игры
bool startGame; // флаг стартового экрана
bool gameOverStart; // флаг выхода из игры со стартового экрана
const int width = 64; // ширина поля
const int height = 32; // высота поля
chtype snake, fruit, nothing, tail; // символы объектов
int x, y, fruitX, fruitY, score; // положение змейки, положение фрукта, счёт
int tailX[100], tailY[100]; // все координаты хвоста
int nTail; // кол-во элементов в хвосте
enum eDirection { STOP = 0, UP, LEFT, DOWN, RIGHT}; // перечисление всех возможных движений
eDirection dir; // переменная на основе перечисления

void setup() { // настройка
	gameOver = false;
	gameExit = false;
	initscr(); // инициализация библиотеки ncurses
	cbreak();
	noecho(); // отключение отображения введенных символов
	halfdelay(2); // ожидание ввода пользователя (по факту скорость змейки)
	scrollok(stdscr, TRUE);
	start_color(); // инициализация цветов символов
	move(0, 0); // перемещение на нулевую координату
	init_pair(1, COLOR_GREEN, COLOR_WHITE); // инициализация пар цвет/фон
	init_pair(2, COLOR_RED, COLOR_WHITE);
	init_pair(3, COLOR_WHITE, COLOR_BLACK);
	init_pair(4, COLOR_BLUE, COLOR_WHITE);
	snake = 'o' | COLOR_PAIR(2); // символ змеи
	tail = 'o' | COLOR_PAIR(1); // символ хвоста
	fruit = 'v' | COLOR_PAIR(2) | A_BLINK; // символ фрукта
	nothing = ' ' | COLOR_PAIR(4); // символ поля
	x = width / 2 - 1; // змейка посередине карты
	y = height / 2 - 1;
	fruitX = rand() % width; // рандомизация положения фрукта
	fruitY = rand() % height;
	score = 0; // обнуление счета
	startGame = true; // поднимаем флаг начального окна
}

void draw() { // отрисовка поля
	refresh(); // каждый вызов очищаем окно
	for (int i = 0; i < height; i++) { // отрисовка поля и элементов
		for (int j = 0; j < width; j++) {
			if (i == y && j == x) { // отрисовка змеи
				move(i, j);
				addch(snake);
			}
			else if (i == fruitY && j == fruitX) { // отрисовка фрукта
				move(i, j);
				addch(fruit);
			}
			else {
				bool print = false; // флаг отрисовки хвоста
				move(i, j);
				for (int k = 0; k < nTail; k++) { // отрисовка хвоста
					if (tailX[k] == j && tailY[k] == i) {
						print = true;
						addch(tail);
					}
				}
				if (!print) // отрисовка поля
					addch(nothing);
			}
		}
	}
	printw("\n\t    Score: %i\n", score); // вывод счета
}

void input() { // ввода пользователя
		switch (getch()) {
			case 'w': // для маленьких клавиш
				dir = UP;
				break;
			case 'a':
				dir = LEFT;
				break;
			case 's':
				dir = DOWN;
				break;
			case 'd':
				dir = RIGHT;
				break;
			case 'x':
				gameOver = true;
				break;
			case 'W': // для больших клавиш
				dir = UP;
				break;
			case 'A':
				dir = LEFT;
				break;
			case 'S':
				dir = DOWN;
				break;
			case 'D':
				dir = RIGHT;
				break;
			case 'X':
				gameOver = true;
				break;
			case 'p':
				dir = STOP;
				break;
		}
}

void logic() { // логика игры

	int prevX = tailX[0]; // положение хвоста по х
	int prevY = tailY[0]; // по у
	int prev2X, prev2Y; // буферные переменные, помещаем следующий элемент в хвосте

	tailX[0] = x; // координаты хвоста изначально равны положению змеи
	tailY[0] = y;

	for (int i = 1; i < nTail; i++) { // обработка всего хвоста
		prev2X = tailX[i]; // записываем текущее положение
		prev2Y = tailY[i];
		tailX[i] = prevX; // записываем предыдущее
		tailY[i] = prevY;
		prevX = prev2X; 
		prevY = prev2Y;
	}

	switch (dir) { // управление змейкой
		case UP:
			y--;
			break;
		case LEFT:
			x--;
			break;
		case DOWN:
			y++;
			break;
		case RIGHT:
			x++;
			break;
	}

	if (x >= width) // обработка перемещения из одного конца поля в другое
		x = 0;
	else if (x < 0)
		x = width - 1;
	if (y >= height)
		y = 0;
	else if (y < 0)
		y = height - 1;

	for (int i = 0; i < nTail; i++) { // проверка на съедание хвоста
		if(tailX[i] == x && tailY[i] == y) 
			gameOver = true;
	}

	if (x == fruitX && y == fruitY) { // обработка поедания фрукта
		score += 10; // увеличение счета
		fruitX = rand() % width; // создание нового фрукта
		fruitY = rand() % height;
		nTail++; // увеличение хвоста на 1
	}

}

void gameEnd() { //обработчик клавиш в конце игры

	switch (getch()) {
		case 'x': // закрытие игры
			gameOver = false;
			gameExit = true;
			exit;
			break;
		case 'X':
			gameOver = false;
			gameExit = true;
			break;
	}

}

void startScreen() { // стартовое окно
		gameOverStart = false;
		move(8, 25);
		switch (getch()) {
			case 'f': // старт игры
				startGame = false;
				break;
			case 'F':
				startGame = false;
				break;
			case 'x': // выход из игры
				gameOverStart = true;
				startGame = false;
				gameExit = true;
				break;
			case 'X':
				gameOverStart = true;
				startGame = false;
				gameExit = true;
				break;
		}
		printw("Target: Fruit\n\n");
		printw("\t\t\t     Keys: \n");
		printw("\t\t\t * wasd - move \n");
		printw("\t\t\t * x - end game \n");
		printw("\t\t\t * p - pause \n\n");
		printw("\t\t\t Press f to start\n");
		printw("\t\t\t Press x to exit");
}

int main() {
	setup(); // вызов настроек
	while(startGame) // вызов стартового экрана
		startScreen();
	while (!gameOver && !startGame && !gameOverStart) { // цикличный вызов пока игра не закончена
		draw();
		input();
		logic();
	}
	if (gameOver) { // окончание игры
		system("clear");
		move(8, 25);
		printw("  Game Over!\n\n");
		printw("\t\t\t    Score: %i\n\n", score);
		printw("\t\t\t Press x to exit\n");
	}
	while (gameOver) {
		gameEnd();
	}
	if (gameExit) { // выход из игры
		endwin();
	}
	return 0;
}
