# Car-gameimport pygame

# Initialize Pygame
pygame.init()

# Screen setup
width = 699
height = 400
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("My Car Game")

# Game control setup
running = True
clock = pygame.time.Clock() # Controls the game speed

print("Start ->")

# Load and scale the car picture
# Note: 'img.png' must be uploaded to the same folder on GitHub as this code!
img = pygame.image.load('img.png')
img = pygame.transform.scale(img, (200, 200))

# Starting position of the car
x = 250
y = 100

# Main Game Loop
while running:
    # 1. Handle Events (like clicking the close window 'X')
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
            
    # 2. Handle Key Presses for Movement
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        x -= 5
    if keys[pygame.K_RIGHT]:
        x += 5
    if keys[pygame.K_UP]:
        y -= 5
    if keys[pygame.K_DOWN]:
        y += 5

    # 3. Drawing and Graphics
    screen.fill((0, 0, 0))     # Clear screen with black background
    screen.blit(img, (x, y))   # Draw the car image at its current x, y position
    pygame.display.update()    # Refresh the screen

    # 4. Keep the game running at a smooth 60 frames per second
    clock.tick(60)

# Clean up and close
pygame.quit()
print("End->")
