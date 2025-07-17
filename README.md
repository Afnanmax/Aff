import pygame
import time
import random
import os

pygame.init()
pygame.mixer.init()

# Sound
eat_sound = pygame.mixer.Sound("eat.wav")
gameover_sound = pygame.mixer.Sound("gameover.wav")

# Colors
white = (255, 255, 255)
black = (0, 0, 0)
red = (200, 30, 30)
green = (0, 255, 0)
blue = (50, 153, 213)
gray = (180, 180, 180)

# Screen
width = 600
height = 400
window = pygame.display.set_mode((width, height))
pygame.display.set_caption("🐍 Snake Game Advanced")

clock = pygame.time.Clock()

# Snake block
block_size = 10

# Fonts
font = pygame.font.SysFont("bahnschrift", 25)
large_font = pygame.font.SysFont("comicsansms", 35)

# Score function
def show_score(score, highscore, level):
    value = font.render(f"Score: {score}  High Score: {highscore}  Level: {level}", True, black)
    window.blit(value, [10, 10])

# Draw snake
def draw_snake(snake_list):
    for block in snake_list:
        pygame.draw.rect(window, green, [block[0], block[1], block_size, block_size])

# Message
def message(msg, color, y_offset=0):
    mesg = large_font.render(msg, True, color)
    rect = mesg.get_rect(center=(width / 2, height / 2 + y_offset))
    window.blit(mesg, rect)

# High score file
def get_highscore():
    with open("highscore.txt", "r") as f:
        return int(f.read())

def save_highscore(score):
    with open("highscore.txt", "w") as f:
        f.write(str(score))

# Game loop
def game_loop():
    x = width / 2
    y = height / 2
    dx, dy = 0, 0

    snake_list = []
    length = 1

    food_x = random.randint(0, (width - block_size) // 10) * 10
    food_y = random.randint(0, (height - block_size) // 10) * 10

    score = 0
    level = 1
    speed = 10
    highscore = get_highscore()

    running = True
    game_over = False

    while running:
        if game_over:
            window.fill(gray)
            message("Game Over!", red)
            message("Press R to Restart or Q to Quit", black, 50)
            pygame.display.update()

            if score > highscore:
                save_highscore(score)
                highscore = score

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_r:
                        game_loop()
                    if event.key == pygame.K_q:
                        running = False

        else:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        dx = -block_size
                        dy = 0
                    elif event.key == pygame.K_RIGHT:
                        dx = block_size
                        dy = 0
                    elif event.key == pygame.K_UP:
                        dy = -block_size
                        dx = 0
                    elif event.key == pygame.K_DOWN:
                        dy = block_size
                        dx = 0

            x += dx
            y += dy

            if x >= width or x < 0 or y >= height or y < 0:
                gameover_sound.play()
                game_over = True

            window.fill(blue)
            pygame.draw.rect(window, red, [food_x, food_y, block_size, block_size])

            head = [x, y]
            snake_list.append(head)
            if len(snake_list) > length:
                del snake_list[0]

            for block in snake_list[:-1]:
                if block == head:
                    gameover_sound.play()
                    game_over = True

            draw_snake(snake_list)
            show_score(score, highscore, level)

            pygame.display.update()

            # Eating food
            if x == food_x and y == food_y:
                eat_sound.play()
                food_x = random.randint(0, (width - block_size) // 10) * 10
                food_y = random.randint(0, (height - block_size) // 10) * 10
                length += 1
                score += 10

                if score % 50 == 0:
                    level += 1
                    speed += 2

            clock.tick(speed)

    pygame.quit()

game_loop()
