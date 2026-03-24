# bubblesgame
import pygame
import random
import math
import sys

# Initialize Pygame
pygame.init()

# Screen dimensions
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Bubbles Game")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 100, 255)
GREEN = (0, 255, 0)
YELLOW = (255, 255, 0)
CYAN = (0, 255, 255)

# Clock for FPS
clock = pygame.time.Clock()
FPS = 60

# Bubble class
class Bubble:
    def __init__(self, x, y, radius, color, speed_x=0, speed_y=0):
        self.x = x
        self.y = y
        self.radius = radius
        self.color = color
        self.speed_x = speed_x
        self.speed_y = speed_y
        self.alive = True

    def update(self):
        # Update position
        self.x += self.speed_x
        self.y += self.speed_y

        # Bounce off walls
        if self.x - self.radius <= 0 or self.x + self.radius >= SCREEN_WIDTH:
            self.speed_x *= -1
            self.x = max(self.radius, min(SCREEN_WIDTH - self.radius, self.x))

        if self.y - self.radius <= 0 or self.y + self.radius >= SCREEN_HEIGHT:
            self.speed_y *= -1
            self.y = max(self.radius, min(SCREEN_HEIGHT - self.radius, self.y))

        # Apply gravity
        self.speed_y += 0.2

    def draw(self, surface):
        pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), self.radius)
        pygame.draw.circle(surface, WHITE, (int(self.x), int(self.y)), self.radius, 2)

    def distance_to(self, other):
        return math.sqrt((self.x - other.x) ** 2 + (self.y - other.y) ** 2)

    def merge(self, other):
        # Merge two bubbles (simple merging logic)
        if self.distance_to(other) < self.radius + other.radius:
            return True
        return False


class Game:
    def __init__(self):
        self.bubbles = []
        self.score = 0
        self.level = 1
        self.font = pygame.font.Font(None, 36)
        self.small_font = pygame.font.Font(None, 24)
        self.spawn_initial_bubbles()

    def spawn_initial_bubbles(self):
        for _ in range(5 + self.level):
            x = random.randint(50, SCREEN_WIDTH - 50)
            y = random.randint(50, SCREEN_HEIGHT - 100)
            radius = random.randint(10, 30)
            color = random.choice([RED, BLUE, GREEN, YELLOW, CYAN])
            speed_x = random.uniform(-3, 3)
            speed_y = random.uniform(-3, 3)
            self.bubbles.append(Bubble(x, y, radius, color, speed_x, speed_y))

    def handle_click(self, pos):
        # Check if click hits any bubble
        for bubble in self.bubbles:
            distance = math.sqrt((bubble.x - pos[0]) ** 2 + (bubble.y - pos[1]) ** 2)
            if distance < bubble.radius:
                bubble.alive = False
                self.score += bubble.radius * 10
                # Spawn smaller bubbles
                if bubble.radius > 10:
                    for _ in range(2):
                        new_radius = bubble.radius // 2
                        new_x = bubble.x + random.randint(-20, 20)
                        new_y = bubble.y + random.randint(-20, 20)
                        new_color = bubble.color
                        speed_x = random.uniform(-5, 5)
                        speed_y = random.uniform(-5, 5)
                        self.bubbles.append(Bubble(new_x, new_y, new_radius, new_color, speed_x, speed_y))

    def update(self):
        # Update all bubbles
        for bubble in self.bubbles:
            bubble.update()

        # Remove dead bubbles
        self.bubbles = [b for b in self.bubbles if b.alive]

        # Check for bubble merging
        for i in range(len(self.bubbles)):
            for j in range(i + 1, len(self.bubbles)):
                if self.bubbles[i].merge(self.bubbles[j]):
                    # Merge bubbles
                    if self.bubbles[i].radius > self.bubbles[j].radius:
                        self.bubbles[i].radius += 1
                        self.bubbles[j].alive = False
                    else:
                        self.bubbles[j].radius += 1
                        self.bubbles[i].alive = False

        self.bubbles = [b for b in self.bubbles if b.alive]

        # Check level completion
        if len(self.bubbles) == 0:
            self.level += 1
            self.spawn_initial_bubbles()

    def draw(self, surface):
        surface.fill(BLACK)

        # Draw bubbles
        for bubble in self.bubbles:
            bubble.draw(surface)

        # Draw score
        score_text = self.font.render(f"Score: {self.score}", True, WHITE)
        surface.blit(score_text, (10, 10))

        # Draw level
        level_text = self.font.render(f"Level: {self.level}", True, WHITE)
        surface.blit(level_text, (10, 50))

        # Draw bubble count
        bubble_text = self.small_font.render(f"Bubbles: {len(self.bubbles)}", True, WHITE)
        surface.blit(bubble_text, (10, 90))

        # Draw instructions
        instructions = self.small_font.render("Click bubbles to pop them!", True, CYAN)
        surface.blit(instructions, (SCREEN_WIDTH - 350, 10))

    def run(self):
        running = True
        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    self.handle_click(event.pos)

            self.update()
            self.draw(screen)
            pygame.display.flip()
            clock.tick(FPS)

        pygame.quit()
        sys.exit()


# Run the game
if __name__ == "__main__":
    game = Game()
    game.run()
