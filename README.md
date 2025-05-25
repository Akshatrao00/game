import pygame
import random

# Initialize
pygame.init()
WIDTH, HEIGHT = 600, 700
win = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Subway Style Runner")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Clock
clock = pygame.time.Clock()
FPS = 60

# Load assets
player_img = pygame.Surface((50, 100))
player_img.fill((0, 255, 0))

obstacle_img = pygame.Surface((50, 100))
obstacle_img.fill((255, 0, 0))

# Lanes
lanes = [150, 275, 400]

class Player:
    def __init__(self):
        self.lane = 1
        self.x = lanes[self.lane]
        self.y = HEIGHT - 120
        self.rect = player_img.get_rect(midbottom=(self.x, self.y + 100))
        self.jump = False
        self.jump_count = 10

    def move_left(self):
        if self.lane > 0:
            self.lane -= 1
            self.x = lanes[self.lane]

    def move_right(self):
        if self.lane < 2:
            self.lane += 1
            self.x = lanes[self.lane]

    def handle_jump(self):
        if self.jump:
            if self.jump_count >= -10:
                neg = 1
                if self.jump_count < 0:
                    neg = -1
                self.rect.y -= (self.jump_count ** 2) * 0.3 * neg
                self.jump_count -= 1
            else:
                self.jump = False
                self.jump_count = 10

    def update(self):
        self.rect.x = self.x
        self.handle_jump()
        win.blit(player_img, self.rect)

class Obstacle:
    def __init__(self):
        self.x = random.choice(lanes)
        self.y = -100
        self.rect = obstacle_img.get_rect(midbottom=(self.x, self.y))

    def update(self):
        self.rect.y += 7
        win.blit(obstacle_img, self.rect)

def main():
    run = True
    player = Player()
    obstacles = []
    score = 0
    font = pygame.font.SysFont(None, 36)

    while run:
        clock.tick(FPS)
        win.fill(WHITE)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            player.move_left()
        if keys[pygame.K_RIGHT]:
            player.move_right()
        if keys[pygame.K_SPACE]:
            if not player.jump:
                player.jump = True

        # Spawn obstacle
        if random.randint(1, 40) == 1:
            obstacles.append(Obstacle())

        # Update obstacles
        for obs in obstacles[:]:
            obs.update()
            if obs.rect.top > HEIGHT:
                obstacles.remove(obs)
                score += 1
            if obs.rect.colliderect(player.rect):
                run = False  # Game Over

        # Update player
        player.update()

        # Show score
        score_text = font.render(f"Score: {score}", True, BLACK)
        win.blit(score_text, (10, 10))

        pygame.display.update()

    pygame.quit()

if __name__ == "__main__":
    main()
