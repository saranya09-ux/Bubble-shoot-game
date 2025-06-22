import pygame
import sys
import math
import random
import time

# Initialize Pygame
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Bubble Shooter Game")
clock = pygame.time.Clock()

# Colors
WHITE = (255, 255, 255)
GREY = (100, 100, 100)
PURPLE = (128, 0, 128)
BALL_COLORS = [(255, 0, 0), (0, 255, 0), (0, 0, 255)]

# Fonts
font = pygame.font.SysFont("Arial", 20)
big_font = pygame.font.SysFont("Arial", 40)

# Buttons (no Angle button)
buttons = {
    "Start": pygame.Rect(50, 540, 80, 40),
    "Restart": pygame.Rect(150, 540, 80, 40),
    "End": pygame.Rect(250, 540, 80, 40),
    "Exchange": pygame.Rect(350, 540, 100, 40)
}

# Game variables
bubble_radius = 15
current_color = random.choice(BALL_COLORS)
shot_bubbles = []
grid = []
message = ""
game_over = False
show_exchange_box = False
score = 0
start_time = time.time()
TIME_LIMIT = 60  # seconds

def create_triangle_grid():
    grid.clear()
    rows = 5
    for row in range(rows):
        row_balls = []
        for col in range(row + 1):
            x = WIDTH // 2 - (row * 35) // 2 + col * 35
            y = 100 + row * 35
            color = random.choice(BALL_COLORS)
            row_balls.append({"x": x, "y": y, "color": color})
        grid.append(row_balls)

def draw_buttons():
    for label, rect in buttons.items():
        pygame.draw.rect(screen, GREY, rect)
        text = font.render(label, True, WHITE)
        screen.blit(text, (rect.x + 10, rect.y + 10))

def draw_exchange_box():
    pygame.draw.rect(screen, GREY, (370, 400, 100, 100))
    for i, color in enumerate(BALL_COLORS):
        pygame.draw.circle(screen, color, (420, 415 + i * 30), 10)

def draw_grid():
    for row in grid:
        for bubble in row:
            pygame.draw.circle(screen, bubble["color"], (bubble["x"], bubble["y"]), bubble_radius)

def draw_score_and_timer():
    elapsed_time = int(time.time() - start_time)
    remaining = max(0, TIME_LIMIT - elapsed_time)
    timer_text = font.render(f"Time: {remaining}s", True, WHITE)
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(timer_text, (10, 10))
    screen.blit(score_text, (WIDTH - 120, 10))
    return remaining

def shoot_bubble_towards(mx, my):
    x, y = 400, 500
    angle = math.atan2(y - my, mx - x)
    dx = math.cos(angle) * 5
    dy = -math.sin(angle) * 5
    shot_bubbles.append({"x": x, "y": y, "dx": dx, "dy": dy, "color": current_color})

def check_collision():
    global message, current_color, score
    for shot in list(shot_bubbles):
        for row in grid:
            for bubble in list(row):
                dist = math.hypot(shot["x"] - bubble["x"], shot["y"] - bubble["y"])
                if dist <= 2 * bubble_radius:
                    if shot["color"] == bubble["color"]:
                        row.remove(bubble)
                        score += 1
                        message = "Well Done!"
                    else:
                        row.append({"x": int(shot["x"]), "y": int(shot["y"]), "color": shot["color"]})
                        message = ""
                    if shot in shot_bubbles:
                        shot_bubbles.remove(shot)
                    current_color = random.choice(BALL_COLORS)
                    return

def check_game_over():
    for row in grid:
        for bubble in row:
            if bubble["y"] >= 480:
                return True
    return False

def drop_balls():
    for row in grid:
        for bubble in row:
            bubble["y"] += 5

def is_grid_empty():
    return all(len(row) == 0 for row in grid)

# Start with initial grid
create_triangle_grid()

# Main Loop
running = True
while running:
    screen.fill(PURPLE)
    draw_buttons()
    draw_grid()
    time_left = draw_score_and_timer()

    if show_exchange_box:
        draw_exchange_box()

    # Gun follows mouse
    mouse_x, mouse_y = pygame.mouse.get_pos()
    angle = math.atan2(500 - mouse_y, mouse_x - 400)
    gun_x = 400 + math.cos(angle) * 40
    gun_y = 500 - math.sin(angle) * 40
    pygame.draw.line(screen, WHITE, (400, 500), (gun_x, gun_y), 5)

    # Draw bubble at gun
    pygame.draw.circle(screen, current_color, (400, 500), bubble_radius)

    # Move and draw shot bubbles
    for shot in list(shot_bubbles):
        shot["x"] += shot["dx"]
        shot["y"] += shot["dy"]
        pygame.draw.circle(screen, shot["color"], (int(shot["x"]), int(shot["y"])), bubble_radius)

    if not game_over:
        check_collision()

        if is_grid_empty():
            message = "CONGRATULATIONS!"
            game_over = True

        elif time_left <= 0:
            message = "TIME OUT!"
            game_over = True

        elif check_game_over():
            message = "GAME OVER"
            game_over = True

    else:
        drop_balls()  # Drop continuously when game is over

    # Display message if any
    if message:
        msg_text = big_font.render(message, True, WHITE)
        screen.blit(msg_text, (WIDTH // 2 - 150, HEIGHT // 2))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        elif event.type == pygame.MOUSEBUTTONDOWN:
            mx, my = pygame.mouse.get_pos()

            if buttons["Start"].collidepoint(mx, my) or buttons["Restart"].collidepoint(mx, my):
                create_triangle_grid()
                score = 0
                shot_bubbles.clear()
                message = ""
                game_over = False
                start_time = time.time()

            elif buttons["End"].collidepoint(mx, my):
                running = False

            elif buttons["Exchange"].collidepoint(mx, my):
                show_exchange_box = not show_exchange_box

            elif show_exchange_box:
                for i, color in enumerate(BALL_COLORS):
                    if 410 < mx < 430 and 405 + i * 30 < my < 435 + i * 30:
                        current_color = color
                        show_exchange_box = False

            elif not game_over:
                shoot_bubble_towards(mx, my)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit(
