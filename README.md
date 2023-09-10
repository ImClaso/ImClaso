import pygame, sys, random
#tạo hàm cho trò chơi
#vẽ sàn
def draw_floor():
    screen.blit(floor,(floor_x_pos,600))
    screen.blit(floor,(floor_x_pos + 432,600))
#tạo ống
def create_pipe():
    random_pipe_pos = random.choice(pipe_height)
    bottom_pipe = pipe_surface.get_rect(midtop = (500,random_pipe_pos))
    top_pipe = pipe_surface.get_rect(midtop = (500,random_pipe_pos -800))
    return bottom_pipe , top_pipe
#di chuyển ống
def move_pipe(pipes):
    for pipe in pipes :
        pipe.centerx -= 5
    return pipes
#vẽ ống
def draw_pipe(pipes):
    for pipe in pipes:
        if pipe.bottom >= 600:
            screen.blit(pipe_surface,pipe)
        else:
            flip_pipe = pygame.transform.flip(pipe_surface,False,True)
            screen.blit(flip_pipe,pipe)
#kiểm tra va chạm
def check_collision(pipes):
    for pipe in pipes:
        if bird_rect.colliderect(pipe):
            hit_sound.play()
            return False
    if bird_rect.top <= -100 or bird_rect.bottom >= 650:
            return False
    return True
#xoay chim
def rotate_bird(bird1):
    new_bird = pygame.transform.rotozoom(bird1,-bird_movement*3,1)
    return new_bird
#hiệu ứng của chim
def bird_animation():
    new_bird = bird_list[bird_index]
    new_bird_rect = new_bird.get_rect(center = (100,bird_rect.centery))
    return new_bird,new_bird_rect
#màn hình điểm
def score_display(game_state):
    if game_state == 'main game':
        score_surface = game_font.render(str(int(score)),True,(255,255,255))
        score_rect = score_surface.get_rect(center = (288,100))
        screen.blit(score_surface,score_rect)
    if game_state == 'game_over':
        score_surface = game_font.render(f'Score: {int(score)}',True,(255,255,255))
        score_rect = score_surface.get_rect(center = (288,130))
        screen.blit(score_surface,score_rect)

        high_score_surface = game_font.render(f'High Score: {int(high_score)}',True,(255,255,255))                          
        high_score_rect = score_surface.get_rect(center = (216,530))
        screen.blit(high_score_surface,high_score_rect)
#viết thông tin support
#def suport(game_state):
#cập nhật điểm
def update_score(score,high_score):
    if score > high_score:
        high_score = score
    return high_score
#tạo màn hình game
pygame.mixer.pre_init(frequency = 44100, size = -16, channels = 2, buffer = 512)
pygame.init()
screen = pygame.display.set_mode((432,700))
clock = pygame.time.Clock()
game_font = pygame.font.Font('dist/04B_19.ttf',40)
#tạo các biến cho trò chơi
gravity = 0.2
bird_movement = 0
game_active = True
score = 0
high_score = 0
#chèn bg
bg = pygame.image.load('dist/background-night.png').convert()
bg = pygame.transform.scale2x(bg)
#chèn sàn
floor = pygame.image.load('dist/floor.png').convert()
floor = pygame.transform.scale2x(floor)
floor_x_pos = 0
#tạo chim
bird_down = pygame.transform.scale2x(pygame.image.load('dist/yellowbird-downflap.png').convert_alpha())
bird_mid = pygame.transform.scale2x(pygame.image.load('dist/yellowbird-midflap.png').convert_alpha())
bird_up = pygame.transform.scale2x(pygame.image.load('dist/yellowbird-upflap.png').convert_alpha())
bird_list = [bird_down,bird_mid,bird_up]
bird_index = 0
bird = bird_list[bird_index]
#tạo timer cho bird
birdflap = pygame.USEREVENT + 1
pygame.time.set_timer(birdflap,200)
#tạo ống
pipe_surface = pygame.image.load('dist/pipe-green.png').convert()
pipe_surface = pygame.transform.scale2x(pipe_surface)
pipe_list = []
bird_rect = bird.get_rect(center = (100,300))
#tạo timer
spawnpipe = pygame.USEREVENT
pygame.time.set_timer(spawnpipe , 1200)
pipe_height = [200,300,400]
#tạo màn hình kết thúc
game_over_surface = pygame.image.load('dist/message.png').convert_alpha()
game_over_rect = game_over_surface.get_rect(center = (216,300))
#chèn âm thanh
flap_sound = pygame.mixer.Sound('dist/sfx_wing.wav')
hit_sound = pygame.mixer.Sound('dist/sfx_hit.wav')
score_sound = pygame.mixer.Sound('dist/sfx_point.wav')
score_sound_countdown = 100
#while loop của game
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and game_active:
                bird_movement = 0
                bird_movement = -10
                flap_sound.play()
            if event.key == pygame.K_SPACE and game_active == False:
                game_active = True
                pipe_list.clear()
                bird_rect.center = (100,300)
                bird_movement = 0
                score = 0
        if event.type == spawnpipe:
            pipe_list.extend(create_pipe())

        if event.type == birdflap:
            if bird_index <2:
                bird_index += 1
            else:
                bird_index = 0
            bird,bird_rect = bird_animation()
    #hiển thị bg
    screen.blit(bg,(0,0))
    #game active
    if game_active: 
        #chim
        bird_movement += gravity
        retated_bird = rotate_bird(bird)
        bird_rect.centery += bird_movement
        screen.blit(retated_bird,bird_rect)
        game_active = check_collision(pipe_list)
        #ống
        pipe_list = move_pipe(pipe_list)
        draw_pipe(pipe_list)
        score += 0.01
        score_display('main game')
        score_sound_countdown -= 1
        if score_sound_countdown <= 0:
            score_sound.play()
            score_sound_countdown = 100
    else:
        screen.blit(game_over_surface,game_over_rect)
        high_score = update_score(score,high_score)
        score_display('game_over')
    #sàn
    floor_x_pos -= 1
    draw_floor()
    if floor_x_pos <= -432:
        floor_x_pos = 0

    pygame.display.update()
    clock.tick(120)
