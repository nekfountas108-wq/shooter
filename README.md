
## PART 5 ADD ON##
from time import time as timer  # imported as timer for convenience


# initialize fonts and mixer
font.init()
mixer.init()

# load fonts
font1 = font.SysFont('Arial', 36)
win_text = font1.render('YOU WIN!', True, (255, 255, 255))
lose_text = font1.render('YOU LOSE!', True, (180, 0, 0))
font2 = font.SysFont('Arial', 36)

# load sounds
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')

# image assets
img_back = "galaxy.jpg"
img_bullet = "bullet.png"
img_hero = "rocket.png"
img_enemy = "ufo.png"


# game variables
score = 0
goal = 20
lost = 0
max_lost = 10



# game window
win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))


class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        super().__init__()
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y

    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))


class Player(GameSprite):
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 80:
            self.rect.x += self.speed

    def fire(self):
        bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top, 15, 20, -15)
        bullets.add(bullet)


class Enemy(GameSprite):
    def update(self):
        self.rect.y += self.speed
        global lost
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost += 1


class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()


# create sprites
ship = Player(img_hero, 5, win_height - 100, 80, 100, 10)

monsters = sprite.Group()
for i in range(5):
    monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    monsters.add(monster)

## PART 5 ADD ON##
img_ast = "asteroid.png"
## PART 5 ADD ON##
life = 3
## PART 5 ADD ON# The same class as enemies but created with different image#
asteroids = sprite.Group()
for i in range(2):
    asteroid = Enemy(img_ast, randint(30, win_width - 30), -40, 80, 50, randint(1, 7))
    asteroids.add(asteroid)

## PART 5 ADD ON##
rel_time = False
num_fire = 0

bullets = sprite.Group()

# game loop variables
finish = False
run = True


while run:
    for e in event.get():
        if e.type == QUIT:
            run = False
        elif e.type == KEYDOWN and e.key == K_SPACE:
            ## PART 5 ADD ON##
            if num_fire < 5 and not rel_time:
                num_fire += 1
                fire_sound.play()
                ship.fire()
            if num_fire >= 5 and not rel_time:
                last_time = timer()
                rel_time = True

    if not finish:
        window.blit(background, (0, 0))
        ship.update()
        monsters.update()

        

        bullets.update()
        bullets.draw(window)

        ship.reset()
        monsters.draw(window)

        ## PART 5 ADD ON##
        asteroids.draw(window)
        ## PART 5 ADD ON##
        asteroids.update()
    
        ## PART 5 ADD ON##
        # reloading
        if rel_time:
            now_time = timer()
            if now_time - last_time < 3:
                reload_text = font2.render('Wait, reload...', True, (150, 0, 0))
                window.blit(reload_text, (260, 460))
            else:
                num_fire = 0
                rel_time = False

        # collision: bullets vs monsters
        collides = sprite.groupcollide(monsters, bullets, True, True)
        for c in collides:
            score += 1
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)

        ## PART 5 ADD ON##
        # collision: player vs enemy/asteroid
        if sprite.spritecollide(ship, monsters, False) or sprite.spritecollide(ship, asteroids, False):
            sprite.spritecollide(ship, monsters, True)
            sprite.spritecollide(ship, asteroids, True)
            life -= 1

        ## PART 5 ADD ON##
        # check loss
        if life == 0 or lost >= max_lost:
            finish = True
            window.blit(lose_text, (200, 200))

        # check win
        if score >= goal:
            finish = True
            window.blit(win_text, (200, 200))

        # score display
        window.blit(font2.render(f"Score: {score}", True, (255, 255, 255)), (10, 20))
        window.blit(font2.render(f"Missed: {lost}", True, (255, 255, 255)), (10, 50))

        ## PART 5 ADD ON##
        # life display
        life_color = (0, 150, 0) if life == 3 else (150, 150, 0) if life == 2 else (150, 0, 0)
        text_life = font1.render(str(life), True, life_color)
        window.blit(text_life, (650, 10))

        display.update()
    else:
        # restart game
        finish = False
        score = 0
        lost = 0

        ## PART 5 ADD ON##
        num_fire = 0
        life = 3

        for a in asteroids:
            a.kill()


        ## PART 5 ADD ON##
        for i in range(2):
            asteroid = Enemy(img_ast, randint(30, win_width - 30), -40, 80, 50, randint(1, 7))
            asteroids.add(asteroid)

        for b in bullets:
            b.kill()
        for m in monsters:
            m.kill()

     
        
        time.delay(3000)
        for i in range(5):
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)
        
    time.delay(50)  
