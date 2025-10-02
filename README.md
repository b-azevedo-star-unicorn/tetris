import pygame
import random
import time

# Inicializa o pygame
pygame.init()

# Cores
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
CYAN = (0, 255, 255)
BLUE = (0, 0, 255)
ORANGE = (255, 165, 0)
YELLOW = (255, 255, 0)
GREEN = (0, 255, 0)
PURPLE = (128, 0, 128)
RED = (255, 0, 0)

# Configurações do jogo
BLOCK_SIZE = 30
GRID_WIDTH = 10
GRID_HEIGHT = 20
SCREEN_WIDTH = BLOCK_SIZE * (GRID_WIDTH + 6)
SCREEN_HEIGHT = BLOCK_SIZE * GRID_HEIGHT

# Formas dos tetrominós
SHAPES = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[1, 1, 1], [0, 1, 0]],  # T
    [[1, 1, 1], [1, 0, 0]],  # J
    [[1, 1, 1], [0, 0, 1]],  # L
    [[0, 1, 1], [1, 1, 0]],  # S
    [[1, 1, 0], [0, 1, 1]]   # Z
]

# Cores dos tetrominós
SHAPE_COLORS = [CYAN, YELLOW, PURPLE, BLUE, ORANGE, GREEN, RED]

class Tetris:
    def __init__(self):
        self.screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
        pygame.display.set_caption("Tetris")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont('Arial', 25)
        self.reset_game()
        
    def reset_game(self):
        self.grid = [[0 for _ in range(GRID_WIDTH)] for _ in range(GRID_HEIGHT)]
        self.current_piece = self.new_piece()
        self.game_over = False
        self.score = 0
        self.level = 1
        self.lines_cleared = 0
        self.fall_time = 0
        self.fall_speed = 0.5  # segundos
        
    def new_piece(self):
        # Escolhe uma forma aleatória
        shape = random.choice(SHAPES)
        color = SHAPE_COLORS[SHAPES.index(shape)]
        
        # Posição inicial (centro do topo)
        x = GRID_WIDTH // 2 - len(shape[0]) // 2
        y = 0
        
        return {'shape': shape, 'x': x, 'y': y, 'color': color}
    
    def valid_move(self, piece, x_offset=0, y_offset=0):
        for y, row in enumerate(piece['shape']):
            for x, cell in enumerate(row):
                if cell:
                    new_x = piece['x'] + x + x_offset
                    new_y = piece['y'] + y + y_offset
                    
                    # Verifica se está dentro dos limites
                    if (new_x < 0 or new_x >= GRID_WIDTH or 
                        new_y >= GRID_HEIGHT):
                        return False
                    
                    # Verifica colisão com peças já fixadas
                    if new_y >= 0 and self.grid[new_y][new_x]:
                        return False
        return True
    
    def rotate_piece(self, piece):
        # Transpõe a matriz e inverte as linhas para rotacionar 90°
        rotated_shape = [list(row) for row in zip(*piece['shape'][::-1])]
        rotated_piece = {'shape': rotated_shape, 'x': piece['x'], 'y': piece['y'], 'color': piece['color']}
        
        # Se a rotação não for válida, não rotaciona
        if self.valid_move(rotated_piece):
            return rotated_piece
        return piece
    
    def lock_piece(self, piece):
        for y, row in enumerate(piece['shape']):
            for x, cell in enumerate(row):
                if cell:
                    # Garante que a peça não está acima do grid
                    if piece['y'] + y >= 0:
                        self.grid[piece['y'] + y][piece['x'] + x] = piece['color']
        
        # Verifica linhas completas
        self.clear_lines()
        
        # Nova peça
        self.current_piece = self.new_piece()
        
        # Verifica game over
        if not self.valid_move(self.current_piece):
            self.game_over = True
    
    def clear_lines(self):
        lines_to_clear = []
        for y, row in enumerate(self.grid):
            if all(row):
                lines_to_clear.append(y)
        
        for line in lines_to_clear:
            # Remove a linha
            del self.grid[line]
            # Adiciona uma nova linha no topo
            self.grid.insert(0, [0 for _ in range(GRID_WIDTH)])
        
        # Atualiza pontuação
        if lines_to_clear:
            self.lines_cleared += len(lines_to_clear)
            self.score += (1, 2, 5, 10)[min(len(lines_to_clear)-1, 3)] * 100 * self.level
            self.level = self.lines_cleared // 10 + 1
            self.fall_speed = max(0.05, 0.5 - (self.level - 1) * 0.05)
    
    def draw_grid(self):
        for y, row in enumerate(self.grid):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(self.screen, cell, 
                                    (x * BLOCK_SIZE, y * BLOCK_SIZE, 
                                     BLOCK_SIZE, BLOCK_SIZE))
                    pygame.draw.rect(self.screen, WHITE, 
                                    (x * BLOCK_SIZE, y * BLOCK_SIZE, 
                                     BLOCK_SIZE, BLOCK_SIZE), 1)
    
    def draw_piece(self, piece):
        for y, row in enumerate(piece['shape']):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(self.screen, piece['color'], 
                                    ((piece['x'] + x) * BLOCK_SIZE, 
                                     (piece['y'] + y) * BLOCK_SIZE, 
                                     BLOCK_SIZE, BLOCK_SIZE))
                    pygame.draw.rect(self.screen, WHITE, 
                                    ((piece['x'] + x) * BLOCK_SIZE, 
                                     (piece['y'] + y) * BLOCK_SIZE, 
                                     BLOCK_SIZE, BLOCK_SIZE), 1)
    
    def draw_info(self):
        # Desenha informações do jogo
        score_text = self.font.render(f'Score: {self.score}', True, WHITE)
        level_text = self.font.render(f'Level: {self.level}', True, WHITE)
        lines_text = self.font.render(f'Lines: {self.lines_cleared}', True, WHITE)
        
        self.screen.blit(score_text, (GRID_WIDTH * BLOCK_SIZE + 10, 30))
        self.screen.blit(level_text, (GRID_WIDTH * BLOCK_SIZE + 10, 60))
        self.screen.blit(lines_text, (GRID_WIDTH * BLOCK_SIZE + 10, 90))
        
        # Desenha instruções
        instructions = [
            "Controls:",
            "← → : Move",
            "↑ : Rotate",
            "↓ : Soft Drop",
            "Space : Hard Drop",
            "P : Pause",
            "R : Restart"
        ]
        
        for i, line in enumerate(instructions):
            text = self.font.render(line, True, WHITE)
            self.screen.blit(text, (GRID_WIDTH * BLOCK_SIZE + 10, 150 + i * 30))
    
    def draw_game_over(self):
        overlay = pygame.Surface((SCREEN_WIDTH, SCREEN_HEIGHT))
        overlay.set_alpha(180)
        overlay.fill(BLACK)
        self.screen.blit(overlay, (0, 0))
        
        game_over_font = pygame.font.SysFont('Arial', 40)
        game_over_text = game_over_font.render('GAME OVER', True, WHITE)
        restart_text = self.font.render('Press R to restart', True, WHITE)
        
        self.screen.blit(game_over_text, 
                        (SCREEN_WIDTH // 2 - game_over_text.get_width() // 2, 
                         SCREEN_HEIGHT // 2 - 50))
        self.screen.blit(restart_text, 
                        (SCREEN_WIDTH // 2 - restart_text.get_width() // 2, 
                         SCREEN_HEIGHT // 2 + 10))
    
    def run(self):
        last_fall_time = time.time()
        paused = False
        
        while True:
            current_time = time.time()
            self.screen.fill(BLACK)
            
            # Eventos
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    return
                
                if event.type == pygame.KEYDOWN:
                    if not self.game_over and not paused:
                        if event.key == pygame.K_LEFT:
                            if self.valid_move(self.current_piece, x_offset=-1):
                                self.current_piece['x'] -= 1
                        elif event.key == pygame.K_RIGHT:
                            if self.valid_move(self.current_piece, x_offset=1):
                                self.current_piece['x'] += 1
                        elif event.key == pygame.K_DOWN:
                            if self.valid_move(self.current_piece, y_offset=1):
                                self.current_piece['y'] += 1
                        elif event.key == pygame.K_UP:
                            self.current_piece = self.rotate_piece(self.current_piece)
                        elif event.key == pygame.K_SPACE:
                            # Hard drop
                            while self.valid_move(self.current_piece, y_offset=1):
                                self.current_piece['y'] += 1
                            self.lock_piece(self.current_piece)
                    
                    if event.key == pygame.K_p:
                        paused = not paused
                    
                    if event.key == pygame.K_r:
                        self.reset_game()
                        paused = False
            
            # Lógica do jogo (se não estiver pausado ou game over)
            if not self.game_over and not paused:
                # Queda automática da peça
                if current_time - last_fall_time > self.fall_speed:
                    if self.valid_move(self.current_piece, y_offset=1):
                        self.current_piece['y'] += 1
                    else:
                        self.lock_piece(self.current_piece)
                    last_fall_time = current_time
            
            # Desenha o grid e a peça atual
            self.draw_grid()
            if not self.game_over:
                self.draw_piece(self.current_piece)
            
            # Desenha informações
            self.draw_info()
            
            # Desenha tela de game over
            if self.game_over:
                self.draw_game_over()
            
            # Desenha texto de pausa
            if paused and not self.game_over:
                pause_font = pygame.font.SysFont('Arial', 40)
                pause_text = pause_font.render('PAUSED', True, WHITE)
                self.screen.blit(pause_text, 
                                (SCREEN_WIDTH // 2 - pause_text.get_width() // 2, 
                                 SCREEN_HEIGHT // 2 - 20))
            
            pygame.display.update()
            self.clock.tick(60)

if __name__ == "__main__":
    game = Tetris()
    game.run()
