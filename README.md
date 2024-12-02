# Robô Seguidor de Linha (Física)

Um robô seguidor de linha é um dispositivo autônomo projetado para seguir um caminho pré-definido, 
geralmente representado por uma linha no chão, utilizando sensores e sistemas de controle. 
Esses robôs são amplamente utilizados para aprendizado em robótica, competições e aplicações práticas, 
como em linhas de produção industriais.

No contexto deste projeto, a proposta é simular o comportamento do robô utilizando Pygame, 
uma biblioteca de Python voltada para desenvolvimento de jogos, e portá-lo para a web com Pygbag, 
um compilador que converte o código em WebAssembly, tornando-o acessível diretamente no navegador.

A simulação visa reproduzir os desafios reais de um robô seguidor de linha, 
considerando que seu comportamento não é tão simples quanto aparenta. 
Um robô seguidor de linha atua com velocidades diferentes em cada roda, 
de acordo com a necessidade da curva que está realizando no momento,
o que também é utilizado em carros reais como um sistema de segurança e prevenção do travamento das rodas.

Para o melhor entendimento de como o robô atua, 
foi colocado um desafio simples de tentar estacionar o mesmo em uma vaga qualquer. 
O desafio pode parecer simples a princípio, porém não é. 
Entender como a velocidade entre as rodas muda a trajetória do robô pode ser bastante desafiador.

## Implementação do Estacionamento e Outros Obstáculos
```python
import pygame
import math
import colors
import configs
import asyncio

pygame.init()
class npc_car:
    def __init__(self,x,y,image):
        self.image = pygame.image.load(image)
        self.x = x
        self.y = y
        self.theta = 270.0
        self.rect = pygame.Rect(0,0,0,0)
        
    def npc_draw(self,display):
        rotated_image = pygame.transform.rotate(self.image, self.theta)
        rect = rotated_image.get_rect(center=(self.x, self.y))
        display.blit(rotated_image, rect.topleft)

npc_cars = [npc_car(80,70,"./assets/images/npc_car1.png"),
            npc_car(230,70,"./assets/images/npc_car.png"),
            npc_car(530,70,"./assets/images/npc_car.png"),
            npc_car(830,70,"./assets/images/npc_car2.png"),
            npc_car(1130,70,"./assets/images/npc_car1.png")]

def reset_game(car,cfg):
    car.x = 200
    car.y = 500
    car.theta = 0
    cfg.trail_rebbot()
    
ACTUAL_STATE_OF_GAME = False

async def main():
    car_size = 7779.52

    cfg = configs.config()
    car = configs.car(0.001 * car_size)
    clock = pygame.time.Clock()

    background = pygame.image.load('./assets/images/Background.png')
    actual_time = pygame.time.get_ticks()
    playing = True 
    time_var = 0
    
    while playing:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                playing = False  # Set 'playing' to False when quitting the game
            car.moving(time_var, event)

        time_var = (pygame.time.get_ticks() - actual_time) / 1000
        actual_time = pygame.time.get_ticks()

        cfg.display.fill(cfg.colors["Black"])
        cfg.display.blit(background, (0, 0))
        
        car.moving(time_var)
        car.draw(cfg.display)
        
        for npc_car in npc_cars: 
            npc_car.npc_draw(cfg.display)
            if(car.rect.colliderect(npc_car.image.get_rect(center=(npc_car.x,npc_car.y)))):
                reset_game(car,cfg)
                print("hey")
                
        cfg.trail((car.x, car.y))
        cfg.write_info(int(car.left_wheel_spd), int(car.right_wheel_spd), car.theta)

        pygame.display.flip()
        clock.tick(120)
        await asyncio.sleep(0)


asyncio.run(main())
