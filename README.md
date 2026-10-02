import pygame
import sys

# --- 配置参数 ---
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
FPS = 60
GRAVITY = 0.8
JUMP_STRENGTH = -15
MOVE_SPEED = 5

# 颜色定义
WHITE = (255, 255, 255)
BLUE = (100, 149, 237)
GREEN = (34, 139, 34)
BROWN = (139, 69, 19)
YELLOW = (255, 215, 0)
RED = (220, 20, 60)

class Player(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((30, 40))
        self.image.fill(RED)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.vel_y = 0
        self.on_ground = False

    def update(self, platforms):
        keys = pygame.key.get_pressed()
        dx = 0
        if keys[pygame.K_LEFT]:
            dx = -MOVE_SPEED
        if keys[pygame.K_RIGHT]:
            dx = MOVE_SPEED

        # 水平移动与碰撞
        self.rect.x += dx
        for platform in platforms:
            if self.rect.colliderect(platform.rect):
                if dx > 0:
                    self.rect.right = platform.rect.left
                elif dx < 0:
                    self.rect.left = platform.rect.right

        # 重力
        self.vel_y += GRAVITY
        self.rect.y += self.vel_y
        self.on_ground = False

        # 垂直移动与碰撞
        for platform in platforms:
            if self.rect.colliderect(platform.rect):
                if self.vel_y > 0:
                    self.rect.bottom = platform.rect.top
                    self.vel_y = 0
                    self.on_ground = True
                elif self.vel_y < 0:
                    self.rect.top = platform.rect.bottom
                    self.vel_y = 0

        # 边界限制
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > SCREEN_WIDTH:
            self.rect.right = SCREEN_WIDTH

    def jump(self):
        if self.on_ground:
            self.vel_y = JUMP_STRENGTH

class Platform(pygame.sprite.Sprite):
    def __init__(self, x, y, width, height):
        super().__init__()
        self.image = pygame.Surface((width, height))
        self.image.fill(BROWN)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

class Coin(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((20, 20), pygame.SRCALPHA)
        pygame.draw.circle(self.image, YELLOW, (10, 10), 10)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

class Goal(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((40, 60))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y

def main():
    pygame.init()
    screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
    pygame.display.set_caption("2D 跑酷游戏")
    clock = pygame.time.Clock()
    font = pygame.font.SysFont("simhei", 36)

    # 创建精灵组
    player = Player(100, 400)
    platforms = pygame.sprite.Group()
    coins = pygame.sprite.Group()
    goals = pygame.sprite.Group()
    all_sprites = pygame.sprite.Group()

    # --- 关卡设计 ---
    # 地面和平台
    platforms.add(Platform(0, 550, 800, 50))      # 地面
    platforms.add(Platform(200, 450, 100, 20))    # 平台1
    platforms.add(Platform(400, 380, 100, 20))    # 平台2
    platforms.add(Platform(600, 300, 100, 20))    # 平台3

    # 金币
    coins.add(Coin(230, 410))
    coins.add(Coin(430, 340))
    coins.add(Coin(630, 260))

    # 终点
    goals.add(Goal(740, 490))

    # 加入所有精灵
    all_sprites.add(player)
    all_sprites.add(platforms)
    all_sprites.add(coins)
    all_sprites.add(goals)

    # 游戏状态
    score = 0
    running = True

    while running:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE or event.key == pygame.K_UP:
                    player.jump()

        # 更新
        player.update(platforms)
        
        # 碰撞检测：收集金币
        collected = pygame.sprite.spritecollide(player, coins, True)
        score += len(collected) * 10

        # 碰撞检测：到达终点
        if pygame.sprite.spritecollide(player, goals, False):
            win_text = font.render("恭喜通关！得分: " + str(score), True, WHITE)
            screen.blit(win_text, (SCREEN_WIDTH//2 - 150, SCREEN_HEIGHT//2))
            pygame.display.flip()
            pygame.time.wait(3000)
            running = False

        # 绘制
        screen.fill(BLUE)
        all_sprites.draw(screen)
        
        # 显示得分
        score_text = font.render("得分: " + str(score), True, WHITE)
        screen.blit(score_text, (10, 10))

        pygame.display.flip()

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
