import pygame
import random

# Ekran boyutları
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600

# Renkler
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)  # Altın rengi

# Pygame'i başlat
pygame.init()
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Mario Benzeri Oyun")

# Ses dosyalarını yükle
pygame.mixer.init()
try:
    jump_sound = pygame.mixer.Sound('jump.wav')
    coin_sound = pygame.mixer.Sound('coin.wav')
    level_up_sound = pygame.mixer.Sound('level_up.wav')
    game_over_sound = pygame.mixer.Sound('game_over.wav')
except pygame.error as e:
    print("Ses dosyaları yüklenemedi:", e)

# Mario karakteri sınıfı
class Mario:
    def _init_(self):
        self.x = 100
        self.y = SCREEN_HEIGHT - 100
        self.width = 50
        self.height = 50
        self.velocity = 0
        self.gravity = 0.8
        self.is_jumping = False

    def draw(self):
        pygame.draw.rect(screen, RED, (self.x, self.y, self.width, self.height))

    def update(self):
        if self.is_jumping:
            self.velocity += self.gravity
            self.y += self.velocity

            if self.y >= SCREEN_HEIGHT - 100:  # Yere inince
                self.y = SCREEN_HEIGHT - 100
                self.is_jumping = False
                self.velocity = 0

    def jump(self):
        if not self.is_jumping:
            self.velocity = -15
            self.is_jumping = True
            jump_sound.play()

# Engeller sınıfı
class Obstacle:
    def _init_(self, x, width, height):
        self.x = x
        self.y = SCREEN_HEIGHT - height
        self.width = width
        self.height = height

    def draw(self):
        pygame.draw.rect(screen, GREEN, (self.x, self.y, self.width, self.height))

    def update(self, speed):
        self.x -= speed

    def off_screen(self):
        return self.x + self.width < 0

# Altın sınıfı (coin)
class Coin:
    def _init_(self, x, y):
        self.x = x
        self.y = y
        self.width = 20
        self.height = 20

    def draw(self):
        pygame.draw.rect(screen, YELLOW, (self.x, self.y, self.width, self.height))

    def update(self, speed):
        self.x -= speed

    def off_screen(self):
        return self.x + self.width < 0

# Seviye bilgisi
current_level = 1
max_level = 5
speed = 5

# Mario ve engeller
mario = Mario()
obstacles = []
coins = []

# Ana döngü
running = True
clock = pygame.time.Clock()
while running:
    screen.fill(WHITE)

    # Olaylar
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                mario.jump()

    # Engelleri güncelle ve ekle
    if random.randint(0, 100) < 3:
        obstacles.append(Obstacle(SCREEN_WIDTH, random.randint(50, 100), random.randint(30, 70)))

    for obstacle in obstacles:
        obstacle.update(speed)
        obstacle.draw()
        if obstacle.off_screen():
            obstacles.remove(obstacle)

    # Altın ekle
    if random.randint(0, 100) < 2:
        coins.append(Coin(SCREEN_WIDTH, random.randint(100, SCREEN_HEIGHT - 100)))

    for coin in coins:
        coin.update(speed)
        coin.draw()
        if coin.off_screen():
            coins.remove(coin)

    # Mario'yu güncelle ve çiz
    mario.update()
    mario.draw()

    # Seviye artırımı
    if len(obstacles) % 10 == 0 and len(obstacles) > 0:
        current_level += 1
        if current_level <= max_level:
            level_up_sound.play()
            speed += 1
        else:
            print("Tüm seviyeleri geçtiniz!")
            running = False

    pygame.display.flip()
    clock.tick(30)

pygame.quit()
