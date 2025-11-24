# final-project
#include <iostream>
#include <vector>
#include <fstream>
#include <string>
using namespace std;

// �ѽL�j�p
const int WIDTH = 10;
const int HEIGHT = 20;

// 7 �ثXù������w�q�]�ϥ� 4x4 �x�}�^
vector<vector<vector<int>>> SHAPES = {
    // I
    {{0,0,0,0},
     {1,1,1,1},
     {0,0,0,0},
     {0,0,0,0}},
    // O
    {{0,1,1,0},
     {0,1,1,0},
     {0,0,0,0},
     {0,0,0,0}},
    // T
    {{0,1,0,0},
     {1,1,1,0},
     {0,0,0,0},
     {0,0,0,0}},
    // L
    {{0,0,1,0},
     {1,1,1,0},
     {0,0,0,0},
     {0,0,0,0}},
    // J
    {{1,0,0,0},
     {1,1,1,0},
     {0,0,0,0},
     {0,0,0,0}},
    // S
    {{0,1,1,0},
     {1,1,0,0},
     {0,0,0,0},
     {0,0,0,0}},
    // Z
    {{1,1,0,0},
     {0,1,1,0},
     {0,0,0,0},
     {0,0,0,0}}
};

// ��r���ন���� shape index
int getShapeIndex(char c) {
    string order = "IOTLJSZ";
    for (int i = 0; i < 7; i++)
        if (order[i] == c) return i;
    return 0;
}

struct Block {
    vector<vector<int>> shape;
    int x, y;
    Block() {}
    Block(char type) {
        shape = SHAPES[getShapeIndex(type)];
        x = 3; y = 0;  // �w�]�ͦ���m
    }
};

// ���� 4x4 ����]���ɰw�^
vector<vector<int>> rotate(vector<vector<int>> s) {
    vector<vector<int>> r(4, vector<int>(4, 0));
    for (int i=0;i<4;i++)
        for (int j=0;j<4;j++)
            r[j][3-i] = s[i][j];
    return r;
}

class Tetris {
private:
    vector<vector<int>> board;
    Block cur;
    int score;
    bool gameOver;

public:
    Tetris() {
        board = vector<vector<int>>(HEIGHT, vector<int>(WIDTH, 0));
        score = 0;
        gameOver = false;
    }

    bool canMove(Block b, int dx, int dy, vector<vector<int>> shape) {
        for (int i=0;i<4;i++)
            for (int j=0;j<4;j++) {
                if (shape[i][j]) {
                    int nx = b.x + j + dx;
                    int ny = b.y + i + dy;
                    if (nx<0 || nx>=WIDTH || ny>=HEIGHT) return false;
                    if (ny>=0 && board[ny][nx]) return false;
                }
            }
        return true;
    }

    void fixBlock(Block b) {
        for (int i=0;i<4;i++)
            for (int j=0;j<4;j++)
                if (b.shape[i][j]) {
                    int x = b.x + j;
                    int y = b.y + i;
                    if (y>=0) board[y][x] = 1;
                }
    }

    void clearLines() {
        int lines = 0;
        for (int i=HEIGHT-1;i>=0;i--) {
            bool full = true;
            for (int j=0;j<WIDTH;j++)
                if (board[i][j]==0) full=false;
            if (full) {
                lines++;
                for (int k=i;k>0;k--)
                    board[k]=board[k-1];
                board[0]=vector<int>(WIDTH,0);
                i++;
            }
        }
        if (lines==1) score+=100;
        else if (lines==2) score+=300;
        else if (lines==3) score+=500;
        else if (lines>=4) score+=800;
    }

    void spawn(char type) {
        cur = Block(type);
        if (!canMove(cur,0,0,cur.shape)) gameOver=true;
    }

    void dropDown() {
        while (canMove(cur,0,1,cur.shape)) cur.y++;
        fixBlock(cur);
        clearLines();
    }

    void command(string cmds) {
        for (char c : cmds) {
            if (c=='A') {
                if (canMove(cur,-1,0,cur.shape)) cur.x--;
            } else if (c=='D') {
                if (canMove(cur,1,0,cur.shape)) cur.x++;
            } else if (c=='R') {
                auto newShape=rotate(cur.shape);
                if (canMove(cur,0,0,newShape)) cur.shape=newShape;
            } else if (c=='F') {
                dropDown();
                return;
            }
        }
        if (canMove(cur,0,1,cur.shape)) cur.y++;
        else {
            fixBlock(cur);
            clearLines();
        }
    }

    void printBoard(ofstream &out) {
        for (int i=0;i<HEIGHT;i++) {
            for (int j=0;j<WIDTH;j++)
                out << (board[i][j] ? '#' : '.');
            out << "\n";
        }
    }

    void playGame() {
        ifstream fin("input.txt");
        ofstream fout("output.txt");
        if (!fin) return;

        int N;
        fin >> N;
        string seq; fin >> seq;

        for (int i=0;i<N && !gameOver;i++) {
            string line;
            if (!(fin >> line)) break;
            spawn(seq[i]);
            command(line);
        }

        fout << score << "\n";
        fout << (gameOver ? "GAME OVER" : "IN PROGRESS") << "\n";
        printBoard(fout);
    }
};

int main() {
    Tetris t;
    t.playGame();
    return 0;
}


