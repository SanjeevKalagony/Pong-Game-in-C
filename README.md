# Pong-Game-in-C using Raylib game engine
#include "raylib.h"
#include "Math.h"

constexpr float SCREEN_WIDTH = 1200.0f;
constexpr float SCREEN_HEIGHT = 800.0f;
constexpr Vector2 CENTER{ SCREEN_WIDTH * 0.5f, SCREEN_HEIGHT * 0.5f };

// Ball can move half the screen width per-second
constexpr float BALL_SPEED = SCREEN_WIDTH * 0.5f;
constexpr float BALL_SIZE = 40.0f;

// Paddles can move half the screen height per-second
constexpr float PADDLE_SPEED = SCREEN_HEIGHT * 0.5f;
constexpr float PADDLE_WIDTH = 40.0f;
constexpr float PADDLE_HEIGHT = 80.0f;

struct Box
{
    float xMin;
    float xMax;
    float yMin;
    float yMax;
};

bool BoxOverlap(Box box1, Box box2)
{
    bool x = box1.xMax >= box2.xMin && box1.xMin <= box2.xMax;
    bool y = box1.yMax >= box2.yMin && box1.yMin <= box2.yMax;
    return x && y;
}

Rectangle BoxToRec(Box box)
{
    Rectangle rec;
    rec.x = box.xMin;
    rec.y = box.yMin;
    rec.width = box.xMax - box.xMin;
    rec.height = box.yMax - box.yMin;
    return rec;
}

Box BallBox(Vector2 position)
{
    Box box;
    box.xMin = position.x - BALL_SIZE * 0.5f;
    box.xMax = position.x + BALL_SIZE * 0.5f;
    box.yMin = position.y - BALL_SIZE * 0.5f;
    box.yMax = position.y + BALL_SIZE * 0.5f; 
    return box;
}

Box PaddleBox(Vector2 position)
{
    Box box;
    box.xMin = position.x - PADDLE_WIDTH * 0.5f;
    box.xMax = position.x + PADDLE_WIDTH * 0.5f;
    box.yMin = position.y - PADDLE_HEIGHT * 0.5f;
    box.yMax = position.y + PADDLE_HEIGHT * 0.5f;
    return box;
}

void ResetBall(Vector2& position, Vector2& direction)
{
    position = CENTER;
    direction.x = rand() % 2 == 0 ? -1.0f : 1.0f;
    direction.y = 0.0f;
    direction = Rotate(direction, Random(0.0f, 360.0f) * DEG2RAD);
}

void DrawBall(Vector2 position, Color color)
{
    Box ballBox = BallBox(position);
    DrawRectangleRec(BoxToRec(ballBox), color);
}

void DrawPaddle(Vector2 position, Color color)
{
    Box paddleBox = PaddleBox(position);
    DrawRectangleRec(BoxToRec(paddleBox), color);
}

int main()
{
    Vector2 ballPosition;
    Vector2 ballDirection;
    ResetBall(ballPosition, ballDirection);

    Vector2 paddle1Position, paddle2Position;
    paddle1Position.x = SCREEN_WIDTH * 0.05f;
    paddle2Position.x = SCREEN_WIDTH * 0.95f;
    paddle1Position.y = paddle2Position.y = CENTER.y;

    int testScore1 = 0;
    int testScore2 = 0;
    int paddleDir = 0;

    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Pong");
    SetTargetFPS(60);
    while (!WindowShouldClose())
    {
        float dt = GetFrameTime();
        float ballDelta = BALL_SPEED * dt;
        float paddleDelta = PADDLE_SPEED * dt;

        // Move paddle with key input
        if (IsKeyDown(KEY_W))
            paddle1Position.y -= paddleDelta;
        if (IsKeyDown(KEY_S))
            paddle1Position.y += paddleDelta;
        // Move paddle with key input
        if (IsKeyDown(KEY_P))
            paddle2Position.y -= paddleDelta;
        if (IsKeyDown(KEY_L))
            paddle2Position.y += paddleDelta;

        float phh = PADDLE_HEIGHT * 0.5f;
        paddle1Position.y = Clamp(paddle1Position.y, phh, SCREEN_HEIGHT - phh);
        paddle2Position.y = Clamp(paddle2Position.y, phh, SCREEN_HEIGHT - phh);

        // Change the ball's direction on-collision
        Vector2 ballPositionNext = ballPosition + ballDirection * ballDelta;
        Box ballBox = BallBox(ballPositionNext);
        Box paddle1Box = PaddleBox(paddle1Position);
        Box paddle2Box = PaddleBox(paddle2Position);

        // TODO -- increment the scoring player's score after they've touched the ball and the ball goes too far right/left
        //testScore++;
        if (ballBox.xMin < 0.0f || ballBox.xMax > SCREEN_WIDTH)
        {
            ballDirection.x *= -1.0f;
            if (paddleDir == 1 && ballBox.xMax > SCREEN_WIDTH)
            {
                testScore1++;
                paddleDir = 0;
            }
            else if (paddleDir == -1 && ballBox.xMin < 0.0f)
            {
                testScore2++;
                paddleDir = 0;
            }
        }
        if (ballBox.yMin < 0.0f || ballBox.yMax > SCREEN_HEIGHT)
        {
            ballDirection.y *= -1.0f;
        }
        if (BoxOverlap(ballBox, paddle1Box) || BoxOverlap(ballBox, paddle2Box))
        {
            ballDirection.x *= -1.0f;
            if (BoxOverlap(ballBox, paddle1Box))
                paddleDir = 1;
            else if (BoxOverlap(ballBox, paddle2Box))
                paddleDir = -1;
        }

        // Update ball position after collision resolution, then render
        ballPosition = ballPosition + ballDirection * ballDelta;

        BeginDrawing();
        ClearBackground(BLACK);
        DrawBall(ballPosition, WHITE);
        DrawPaddle(paddle1Position, WHITE);
        DrawPaddle(paddle2Position, WHITE);

        // Text format requires you to put a '%i' wherever you want an integer, then add said integer after the comma
        const char* testScoreText1 = TextFormat("Test Score 1: %i ", testScore1);
        const char* testScoreText2 = TextFormat("Test Score 2: %i ", testScore2);
        // We can measure our text for more exact positioning. This puts our score in the center of our screen!
        DrawText(testScoreText1, SCREEN_WIDTH * 0.1f + MeasureText(testScoreText1, 20) * 0.0f, 50, 20, BLUE);
        DrawText(testScoreText2, SCREEN_WIDTH * 0.9f - MeasureText(testScoreText2, 20) * 1.0f, 50, 20, BLUE);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
