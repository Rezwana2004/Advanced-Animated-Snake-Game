import pygame
import random
import math

# ---------------- INITIAL SETUP ---------------- #
pygame.init()
pygame.mixer.init()

# ---------------- SCREEN ---------------- #
WIDTH = 800
HEIGHT = 600

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Advanced Animated Snake Game")

clock = pygame.time.Clock()

# ---------------- COLORS ---------------- #
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 100)
RED = (255, 50, 50)
BLUE = (40, 40, 80)
YELLOW = (255, 255, 0)
PURPLE = (170, 0, 255)

# ---------------- FONTS ---------------- #
font = pygame.font.SysFont("arial", 28)
big_font = pygame.font.SysFont("arial", 50)

# ---------------- GAME VARIABLES ---------------- #
BLOCK = 20
INITIAL_SPEED = 10

# ---------------- SOUND ---------------- #
# You can replace with your own .wav files
try:
    eat_sound = pygame.mixer.Sound("eat.wav")
    gameover_sound = pygame.mixer.Sound("gameover.wav")
except:
    eat_sound = None
    gameover_sound = None

# ---------------- PARTICLES ---------------- #
particles = []

class Particle:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.radius = random.randint(2, 5)
        self.speed_x = random.randint(-3, 3)
        self.speed_y = random.randint(-3, 3)
        self.life = 30

    def update(self):
        self.x += self.speed_x
        self.y += self.speed_y
        self.life -= 1

    def draw(self):
        pygame.draw.circle(screen, YELLOW, (int(self.x), int(self.y)), self.radius)

# ---------------- FUNCTIONS ---------------- #
def draw_text(text, color, x, y, font_type=font):
    img = font_type.render(text, True, color)
    screen.blit(img, (x, y))


def draw_background():
    time_now = pygame.time.get_ticks() / 1000

    r = int((math.sin(time_now) + 1) * 40)
    g = int((math.sin(time_now + 2) + 1) * 40)
    b = int((math.sin(time_now + 4) + 1) * 40)

    screen.fill((r, g, b))


def draw_food(food_x, food_y):
    pulse = abs(math.sin(pygame.time.get_ticks() * 0.005))
    size = int(BLOCK + pulse * 8)

    pygame.draw.rect(
        screen,
        RED,
        [food_x - size // 4, food_y - size // 4, size, size],
        border_radius=8
    )


def draw_snake(snake):
    for i, segment in enumerate(snake):

        color_intensity = 255 - (len(snake) - i) * 3
        color_intensity = max(color_intensity, 50)

        pygame.draw.rect(
            screen,
            (0, color_intensity, 0),
            [segment[0], segment[1], BLOCK, BLOCK],
            border_radius=5
        )

    # Snake eyes
    head = snake[-1]

    pygame.draw.circle(screen, WHITE, (head[0] + 5, head[1] + 5), 3)
    pygame.draw.circle(screen, WHITE, (head[0] + 15, head[1] + 5), 3)


def create_food():
    x = random.randrange(0, WIDTH - BLOCK, BLOCK)
    y = random.randrange(0, HEIGHT - BLOCK, BLOCK)
    return x, y


def game_over_screen(score):
    if gameover_sound:
        gameover_sound.play()

    for i in range(3):
        screen.fill(RED)
        pygame.display.update()
        pygame.time.delay(150)

        screen.fill(BLACK)
        pygame.display.update()
        pygame.time.delay(150)

    while True:
        screen.fill(BLACK)

        draw_text("GAME OVER", RED, WIDTH // 2 - 150, HEIGHT // 3, big_font)
        draw_text(f"Final Score: {score}", WHITE, WIDTH // 2 - 100, HEIGHT // 2)
        draw_text("Press R to Restart or Q to Quit", YELLOW, WIDTH // 2 - 180, HEIGHT // 2 + 60)

        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    game()
                if event.key == pygame.K_q:
                    pygame.quit()
                    quit()


# ---------------- MAIN GAME ---------------- #
def game():

    snake = [[100, 100]]
    snake_length = 1

    x_change = BLOCK
    y_change = 0

    food_x, food_y = create_food()

    speed = INITIAL_SPEED
    score = 0
    level = 1

    running = True

    while running:

        # ---------------- EVENTS ---------------- #
        for event in pygame.event.get():

            if event.type == pygame.QUIT:
                running = False

            if event.type == pygame.KEYDOWN:

                if event.key == pygame.K_LEFT and x_change == 0:
                    x_change = -BLOCK
                    y_change = 0

                elif event.key == pygame.K_RIGHT and x_change == 0:
                    x_change = BLOCK
                    y_change = 0

                elif event.key == pygame.K_UP and y_change == 0:
                    x_change = 0
                    y_change = -BLOCK

                elif event.key == pygame.K_DOWN and y_change == 0:
                    x_change = 0
                    y_change = BLOCK

        # ---------------- MOVE ---------------- #
        head_x = snake[-1][0] + x_change
        head_y = snake[-1][1] + y_change

        new_head = [head_x, head_y]
        snake.append(new_head)

        if len(snake) > snake_length:
            del snake[0]

        # ---------------- WALL COLLISION ---------------- #
        if (
            head_x < 0 or
            head_x >= WIDTH or
            head_y < 0 or
            head_y >= HEIGHT
        ):
            game_over_screen(score)

        # ---------------- SELF COLLISION ---------------- #
        for segment in snake[:-1]:
            if segment == new_head:
                game_over_screen(score)

        # ---------------- FOOD COLLISION ---------------- #
        if head_x == food_x and head_y == food_y:

            if eat_sound:
                eat_sound.play()

            food_x, food_y = create_food()

            snake_length += 1
            score += 10

            # PARTICLE EFFECT
            for _ in range(15):
                particles.append(Particle(food_x, food_y))

            # LEVEL SYSTEM
            if score % 50 == 0:
                level += 1
                speed += 2

        # ---------------- DRAW ---------------- #
        draw_background()

        # FOOD
        draw_food(food_x, food_y)

        # PARTICLES
        for particle in particles[:]:
            particle.update()
            particle.draw()

            if particle.life <= 0:
                particles.remove(particle)

        # SNAKE
        draw_snake(snake)

        # UI
        draw_text(f"Score: {score}", WHITE, 10, 10)
        draw_text(f"Level: {level}", YELLOW, 10, 45)
        draw_text(f"Speed: {speed}", GREEN, 10, 80)

        pygame.display.update()

        clock.tick(speed)

    pygame.quit()
    quit()


# ---------------- START SCREEN ---------------- #
def start_screen():

    while True:

        draw_background()

        title_y = 150 + math.sin(pygame.time.get_ticks() * 0.005) * 10

        draw_text(
            "ADVANCED SNAKE GAME",
            GREEN,
            WIDTH // 2 - 220,
            title_y,
            big_font
        )

        draw_text(
            "Press SPACE to Start",
            WHITE,
            WIDTH // 2 - 130,
            HEIGHT // 2
        )

        draw_text(
            "Arrow Keys to Move",
            YELLOW,
            WIDTH // 2 - 120,
            HEIGHT // 2 + 50
        )

        pygame.display.update()

        for event in pygame.event.get():

            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    game()


# ---------------- RUN GAME ---------------- #
start_screen()
