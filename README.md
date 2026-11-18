Tentu, kita ubah struktur kodenya menjadi berbasis Object-Oriented Programming (OOP) agar lebih modular, rapi, dan mudah dikembangkan.

Dalam pendekatan OOP ini, kita membaginya menjadi 3 kelas utama:

Snake: Mengatur posisi tubuh ular, pergerakan, pertumbuhan, dan pengecekan tabrakan.

Food: Mengatur kemunculan dan posisi makanan secara acak.

Game: Mengatur loop utama game, input keyboard, render layar, dan status game (game over).
import pygame
import random
import time

# Inisialisasi Pygame
pygame.init()

# Definisi Warna (RGB)
WHITE = (255, 255, 255)
YELLOW = (255, 255, 102)
BLACK = (0, 0, 0)
RED = (213, 50, 80)
GREEN = (0, 255, 0)
BLUE = (50, 153, 213)

class Food:
    def __init__(self, width, height, block_size):
        self.width = width
        self.height = height
        self.block_size = block_size
        self.x = 0
        self.y = 0
        self.spawn()

    def spawn(self):
        """Membuat posisi makanan baru secara acak"""
        self.x = round(random.randrange(0, self.width - self.block_size) / self.block_size) * self.block_size
        self.y = round(random.randrange(0, self.height - self.block_size) / self.block_size) * self.block_size

    def draw(self, surface):
        """Menggambar makanan di layar"""
        pygame.draw.rect(surface, YELLOW, [self.x, self.y, self.block_size, self.block_size])


class Snake:
    def __init__(self, width, height, block_size):
        self.block_size = block_size
        self.reset(width, height)

    def reset(self, width, height):
        """Mengatur ulang posisi awal ular saat game dimulai/diulang"""
        self.x = width / 2
        self.y = height / 2
        self.x_change = 0
        self.y_change = 0
        self.body = []
        self.length = 1

    def update(self):
        """Memperbarui posisi kepala dan tubuh ular"""
        self.x += self.x_change
        self.y += self.y_change
        
        snake_head = [self.x, self.y]
        self.body.append(snake_head)
        
        if len(self.body) > self.length:
            del self.body[0]

    def change_direction(self, dx, dy):
        """Mengubah arah gerak ular (mencegah ular berbalik arah langsung ke tubuh sendiri)"""
        if (dx == -self.block_size and self.x_change == self.block_size) or \
           (dx == self.block_size and self.x_change == -self.block_size) or \
           (dy == -self.block_size and self.y_change == self.block_size) or \
           (dy == self.block_size and self.y_change == -self.block_size):
            return
        self.x_change = dx
        self.y_change = dy

    def check_boundary_collision(self, width, height):
        """Mengecek apakah ular menabrak tembok"""
        return self.x >= width or self.x < 0 or self.y >= height or self.y < 0

    def check_self_collision(self):
        """Mengecek apakah ular menabrak tubuhnya sendiri"""
        head = self.body[-1]
        for segment in self.body[:-1]:
            if segment == head:
                return True
        return False

    def draw(self, surface):
        """Menggambar seluruh bagian tubuh ular"""
        for segment in self.body:
            pygame.draw.rect(surface, GREEN, [segment[0], segment[1], self.block_size, self.block_size])


class Game:
    def __init__(self):
        self.width = 600
        self.height = 400
        self.display = pygame.display.set_mode((self.width, self.height))
        pygame.display.set_caption('Game Ular OOP - Python')
        
        self.clock = pygame.time.Clock()
        self.block_size = 10
        self.speed = 15
        
        # Font game
        self.font_style = pygame.font.SysFont("bahnschrift", 25)
        self.score_font = pygame.font.SysFont("comicsansms", 20)
        
        # Inisialisasi Objek Ular dan Makanan
        self.snake = Snake(self.width, self.height, self.block_size)
        self.food = Food(self.width, self.height, self.block_size)

    def display_score(self):
        """Menampilkan skor pemain"""
        value = self.score_font.render("Skor: " + str(self.snake.length - 1), True, YELLOW)
        self.display.blit(value, [10, 10])

    def show_message(self, msg, color):
        """Menampilkan pesan teks di layar (misal: Game Over)"""
        mesg = self.font_style.render(msg, True, color)
        self.display.blit(mesg, [self.width / 6, self.height / 3])

    def run(self):
        """Loop utama jalannya permainan"""
        game_over = False
        game_close = False

        while not game_over:
            while game_close:
                self.display.fill(BLUE)
                self.show_message("Game Over! Tekan C (Main Lagi) atau Q (Keluar)", RED)
                self.display_score()
                pygame.display.update()

                for event in pygame.event.get():
                    if event.type == pygame.KEYDOWN:
                        if event.key == pygame.K_q:
                            game_over = True
                            game_close = False
                        if event.key == pygame.K_c:
                            self.snake.reset(self.width, self.height)
                            self.food.spawn()
                            self.run()
                            return

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    game_over = True
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        self.snake.change_direction(-self.block_size, 0)
                    elif event.key == pygame.K_RIGHT:
                        self.snake.change_direction(self.block_size, 0)
                    elif event.key == pygame.K_UP:
                        self.snake.change_direction(0, -self.block_size)
                    elif event.key == pygame.K_DOWN:
                        self.snake.change_direction(0, self.block_size)

            # Perbarui posisi ular
            self.snake.update()

            # Cek kondisi kalah (tabrak dinding atau badan sendiri)
            if self.snake.check_boundary_collision(self.width, self.height) or self.snake.check_self_collision():
                game_close = True

            # Cek jika ular makan makanan
            if self.snake.x == self.food.x and self.snake.y == self.food.y:
                self.food.spawn()
                self.snake.length += 1

            # Render grafis ke layar
            self.display.fill(BLUE)
            self.food.draw(self.display)
            self.snake.draw(self.display)
            self.display_score()

            pygame.display.update()
            self.clock.tick(self.speed)

        pygame.quit()
        quit()


if __name__ == "__main__":
    game_app = Game()
    game_app.run()