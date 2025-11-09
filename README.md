import pygame
import random

# 1. Initialize Pygame
pygame.init()

# --- Game Constants ---
SCREEN_WIDTH = 500
SCREEN_HEIGHT = 700
SCREEN = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Pathaan Flap!")

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
BLUE = (0, 0, 255) # Pathaan color

# --- Pathaan Player Properties (आपका किरदार) ---
player_x = 50
player_y = 350
player_size = 30
player_velocity = 0
gravity = 1

# --- Obstacle (Drone Blade) Properties (रुकावटें) ---
pipe_speed = 5
pipe_gap = 200 # The safe space for Pathaan to fly through
pipe_width = 50
pipes = [] # List to store all pipe pairs

# Function to create a new pipe pair
def create_pipe():
    # Randomly determine the height of the gap
    pipe_height = random.randint(100, SCREEN_HEIGHT - pipe_gap - 100)
    
    # Upper pipe (Drone Blade Top)
    upper_pipe = pygame.Rect(SCREEN_WIDTH, 0, pipe_width, pipe_height)
    
    # Lower pipe (Drone Blade Bottom)
    lower_pipe = pygame.Rect(SCREEN_WIDTH, pipe_height + pipe_gap, pipe_width, SCREEN_HEIGHT - pipe_height - pipe_gap)
    
    pipes.append((upper_pipe, lower_pipe, False)) # False means not yet scored

# --- Game Loop ---
running = True
clock = pygame.time.Clock()
score = 0
font = pygame.font.Font(None, 36)

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        # Tap to Flap Mechanic (Pathaan's jump)
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                player_velocity = -12 # Jump strength

    # 1. Apply Gravity (गिरना)
    player_velocity += gravity
    player_y += player_velocity
    
    # Create new pipes (Drone Blades) every 90 frames
    if len(pipes) == 0 or pipes[-1][0].left < SCREEN_WIDTH - 200:
        create_pipe()

    # 2. Move and Draw Pipes (रुकावटें आगे बढ़ाना)
    for pipe_pair in pipes:
        pipe_pair[0].x -= pipe_speed
        pipe_pair[1].x -= pipe_speed
        
        # Drawing the obstacle
        pygame.draw.rect(SCREEN, BLACK, pipe_pair[0]) 
        pygame.draw.rect(SCREEN, BLACK, pipe_pair[1])

    # Remove off-screen pipes
    pipes = [pipe for pipe in pipes if pipe[0].right > 0]

    # 3. Collision and Scoring (टकराव और स्कोर)
    player_rect = pygame.Rect(player_x, player_y, player_size, player_size)
    
    # Check for ground/ceiling collision
    if player_y > SCREEN_HEIGHT - player_size or player_y < 0:
        running = False # Game Over

    # Check for pipe collision (Drone Blades hit)
    for pipe_pair in pipes:
        if player_rect.colliderect(pipe_pair[0]) or player_rect.colliderect(pipe_pair[1]):
            running = False # Game Over
            
        # Scoring (10 Points for passing through)
        if pipe_pair[0].right < player_x and not pipe_pair[2]:
            score += 10
            pipe_pair = (pipe_pair[0], pipe_pair[1], True) # Mark as scored

    # 4. Drawing (سب کچھ اسکرین پر دکھانا)
    SCREEN.fill(WHITE) # White background (You can add a background image later)
    pygame.draw.rect(SCREEN, BLUE, player_rect) # Draw Pathaan (Blue Square)
    
    # Redraw pipes on top of the background
    for pipe_pair in pipes:
        pygame.draw.rect(SCREEN, BLACK, pipe_pair[0]) 
        pygame.draw.rect(SCREEN, BLACK, pipe_pair[1])

    # Display Score
    score_text = font.render(f"Score: {score}", True, BLACK)
    SCREEN.blit(score_text, (10, 10))

    # Update the display
    pygame.display.flip()
    
    # Control the game speed (60 frames per second)
    clock.tick(60)

pygame.quit()
print(f"Game Over! Final Score: {score}")
