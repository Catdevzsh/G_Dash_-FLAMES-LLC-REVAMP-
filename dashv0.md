import pygame
import sys
from array import array

# Initialize Pygame and its mixer
pygame.init()
pygame.mixer.init(frequency=22050, size=-16, channels=2, buffer=512)

# Screen dimensions and setup
SCREEN_WIDTH, SCREEN_HEIGHT = 600, 400
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Geometry Dash by ChatGPT [20XX]")

# Colors
BLUE = (0, 0, 255)
WHITE = (255, 255, 255)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)

# Game variables
player_size = 40
player_x, player_y = 50, SCREEN_HEIGHT - player_size - 10
player_speed = 0
gravity = 1
jump_speed = -15
ground_height = 10
score = 0
game_over = False
game_started = False

# Obstacles
obstacle_width, obstacle_height = 20, 40
obstacle_x = SCREEN_WIDTH
obstacle_speed = 5

# Game loop control
clock = pygame.time.Clock()
running = True
jumping = False

# Font
font = pygame.font.Font(None, 36)

# Function to generate beep sounds with varying frequencies
def generate_beep_sound(frequency=440, duration=0.1):
    sample_rate = pygame.mixer.get_init()[0]
    max_amplitude = 2 ** (abs(pygame.mixer.get_init()[1]) - 1) - 1
    samples = int(sample_rate * duration)
    wave = [int(max_amplitude * ((i // (sample_rate // frequency)) % 2)) for i in range(samples)]
    sound = pygame.mixer.Sound(buffer=array('h', wave))
    sound.set_volume(0.1)
    return sound

# Create a beep sound for jumping
jump_sound = generate_beep_sound(440, 0.1)  # A4

def reset_game():
    global player_x, player_y, player_speed, obstacle_x, score, game_over, jumping, game_started
    player_x, player_y = 50, SCREEN_HEIGHT - player_size - 10
    player_speed = 0
    obstacle_x = SCREEN_WIDTH
    score = 0
    game_over = False
    jumping = False
    game_started = False

def handle_input(event):
    global running, player_speed, jumping, game_over, game_started
    if event.type == pygame.QUIT:
        running = False
    if event.type == pygame.KEYDOWN:
        if event.key in [pygame.K_z, pygame.K_SPACE] and not game_started:
            game_started = True
            if game_over:
                reset_game()
        elif event.key == pygame.K_SPACE and not jumping and not game_over:
            player_speed = jump_speed
            jumping = True
            # Play the beep sound when the player jumps
            jump_sound.play()
        elif event.key == pygame.K_y and game_over:
            reset_game()
        elif event.key == pygame.K_n and game_over:
            running = False

def update_game_state():
    global player_y, player_speed, jumping, obstacle_x, score, game_over
    if game_started and not game_over:
        # Apply gravity
        player_speed += gravity
        player_y += player_speed

        # Check for landing
        if player_y >= SCREEN_HEIGHT - player_size - ground_height:
            player_y = SCREEN_HEIGHT - player_size - ground_height
            player_speed = 0
            jumping = False

        # Move obstacle
        obstacle_x -= obstacle_speed
        if obstacle_x < -obstacle_width:
            obstacle_x = SCREEN_WIDTH
            score += 1  # Increase score when an obstacle passes

        # Check for collision
        if (obstacle_x < player_x + player_size < obstacle_x + obstacle_width or
            obstacle_x < player_x < obstacle_x + obstacle_width) and \
            player_y + player_size > SCREEN_HEIGHT - obstacle_height - ground_height:
            game_over = True

def draw_game():
    screen.fill(BLUE)
    if not game_started:
        # Display start screen
        start_text = font.render("Press Z or Space to Start", True, WHITE)
        start_text_rect = start_text.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2))
        screen.blit(start_text, start_text_rect)
    else:
        pygame.draw.rect(screen, YELLOW, (player_x, player_y, player_size, player_size))
        pygame.draw.rect(screen, BLACK, (obstacle_x, SCREEN_HEIGHT - obstacle_height - ground_height, obstacle_width, obstacle_height))
        pygame.draw.rect(screen, WHITE, (0, SCREEN_HEIGHT - ground_height, SCREEN_WIDTH, ground_height))

        # Display score
        score_text = font.render(f"Score: {score}", True, WHITE)
        screen.blit(score_text, (10, 10))

        # Game over message
        if game_over:
            game_over_text = font.render("Game Over! Press Y to restart or N to quit.", True, WHITE)
            screen.blit(game_over_text, (50, SCREEN_HEIGHT // 2))

    # Update display
    pygame.display.flip()

# Main game loop
while running:
    for event in pygame.event.get():
        handle_input(event)

    update_game_state()
    draw_game()
    clock.tick(30)

# Quit Pygame
pygame.quit()
sys.exit()
