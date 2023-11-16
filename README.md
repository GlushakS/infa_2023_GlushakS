# infa_2023_GlushakS
import math
from random import choice, randint

import pygame

FPS = 30

RED = 0xFF0000
BLUE = 0x0000FF
YELLOW = 0xFFC91F
GREEN = 0x00FF00
MAGENTA = 0xFF03B8
CYAN = 0x00FFCC
BLACK = (0, 0, 0)
WHITE = 0xFFFFFF
GREY = 0x7D7D7D
GAME_COLORS = [RED, BLUE, YELLOW, GREEN, MAGENTA, CYAN]

WIDTH = 800
HEIGHT = 600

class Ball:
    def __init__(self, screen: pygame.Surface, x=40, y=450):
        self.screen = screen
        self.x = x
        self.y = y
        self.r = 10
        self.vx = 0
        self.vy = 0
        self.color = choice(GAME_COLORS)
        self.live = 30

    def move(self):
        gravity_x = 0
        gravity_y = 3
        if self.x > WIDTH:
            self.vx = -self.vx
            self.vx -= gravity_x
        if self.y > HEIGHT:
            self.vx -= 0.1 * self.vx
            self.vy = -self.vy
        self.vy -= gravity_y + 0.03*self.vy
        self.vx -= 0.03*self.vx
        self.x += self.vx
        self.y -= self.vy

    def draw(self):
        pygame.draw.circle(
            self.screen,
            self.color,
            (self.x, self.y),
            self.r
        )

    def hittest(self, obj):
        if ((self.x - obj.x) ** 2 + (self.y - obj.y) ** 2) ** 0.5 <= (self.r + obj.r):
            return True
        return False

class Gun:
    def __init__(self, screen):
        self.screen = screen
        self.f2_power = 10
        self.f2_on = 0
        self.an = 1
        self.color = GREY

    def fire2_start(self, event):
        self.f2_on = 1

    def fire2_end(self, event):
        global balls, bullet
        bullet += 1
        new_ball = Ball(self.screen)
        new_ball.r += 5
        self.an = math.atan2((event.pos[1]-new_ball.y), (event.pos[0]-new_ball.x))
        new_ball.vx = self.f2_power * math.cos(self.an)
        new_ball.vy = - self.f2_power * math.sin(self.an)
        balls.append(new_ball)
        self.f2_on = 0
        self.f2_power = 20

    def targetting(self, event):
        if event:
            self.an = math.atan((event.pos[1]-450) / (event.pos[0]-20))
        if self.f2_on:
            self.color = RED
        else:
            self.color = GREY

    def draw(self):
        gun_surface = pygame.Surface((60 + 2 * self.f2_power, 10))
        gun_surface.fill(WHITE)
        pygame.draw.line(
            gun_surface,
            self.color,
            (0, 5),
            (30 + self.f2_power, 5),
            5
        )
        gun_surface_rotated = pygame.transform.rotozoom(
            gun_surface.convert_alpha(),
            180 - math.degrees(self.an),
            1
        )
        self.screen.blit(gun_surface_rotated, (-30 - self.f2_power, 440, 30 + self.f2_power, 450))

    def power_up(self):
        if self.f2_on:
            if self.f2_power < 100:
                self.f2_power += 1
            self.color = RED
        else:
            self.color = GREY

class Target:
    def __init__(self):
        self.points = 0
        self.live = 1
        self.new_target()
        self.x = randint(300, 750)
        self.y = randint(200, 550)
        self.r = randint(5, 50)
        self.color = RED
        self.vx = (randint(-5, 5))*(self.r)/20
        self.vy = (randint(-5, 5))*(self.r)/20

    def new_target(self):
        self.live = 1
        x = self.x = randint(300, 750)
        y = self.y = randint(200, 550)
        r = self.r = randint(7, 50)
        color = self.color = RED

    def hit(self, points=1):
        self.points += points

    def move(self):
        if 0 >= self.x - self.r or self.x + self.r >= WIDTH:
            self.vx = -self.vx
        if 0 >= self.y - self.r or self.y + self.r >= HEIGHT:
            self.vy = -self.vy
        self.x += self.vx
        self.y += self.vy

    def draw(self):
        pygame.draw.circle(
            screen,
            self.color,
            (self.x, self.y),
            self.r
        )

pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
bullet = 0
balls = []

clock = pygame.time.Clock()
gun = Gun(screen)
target1 = Target()
target2 = Target()
finished = False

while not finished:
    screen.fill(WHITE)
    gun.draw()
    target1.draw()
    target2.draw()
    for b in balls:
        b.draw()
    pygame.display.update()

    clock.tick(FPS)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            finished = True
        elif event.type == pygame.MOUSEBUTTONDOWN:
            gun.fire2_start(event)
        elif event.type == pygame.MOUSEBUTTONUP:
            gun.fire2_end(event)
        elif event.type == pygame.MOUSEMOTION:
            gun.targetting(event)

    for b in balls:
        b.move()
        if b.hittest(target1) and target1.live:
            target1.live = 0
            target1.hit()
            target1.new_target()
        if b.hittest(target2) and target2.live:
            target2.live = 0
            target2.hit()
            target2.new_target()

    for t in [target1, target2]:
        t.move()

    gun.power_up()

pygame.quit()
