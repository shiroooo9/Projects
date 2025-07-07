# Towers of Hanoi
import pygame
import sys
import time
import random
from MainMenu import MainMenu

class TowersOfHanoiGame:
    def __init__(self):
        pygame.init()

        # Screen settings
        self.width = 800
        self.height = 600
        self.screen = pygame.display.set_mode((self.width, self.height))
        pygame.display.set_caption("Towers of Hanoi")

        # Load and scale background image
        try:
            bg_image = pygame.image.load("night_bg.gif")
            self.background = pygame.transform.scale(bg_image, (self.width, self.height))
            # Darken the background image
            dark_overlay = pygame.Surface((self.width, self.height))
            dark_overlay.set_alpha(100)  # Adjust alpha for darkness level
            dark_overlay.fill((0, 0, 0))
            self.background.blit(dark_overlay, (0, 0))
        except:
            # Fallback to solid color if image loading fails
            self.background = pygame.Surface((self.width, self.height))
            self.background.fill((0, 0, 0))

        # Colors
        self.bg_color = (0, 0, 0)  # Used for UI elements
        self.text_color = (255, 255, 255)  # White text
        self.pole_color = (255, 255, 255)  # White poles
        self.disk_colors = [
            (255, 0, 0),  # Red
            (255, 165, 0),  # Orange
            (255, 219, 88),  # Yellow Mustard
            (0, 128, 0),  # Green
            (0, 0, 255),  # Blue
        ]

        # Game variables
        self.num_disks = 5
        self.poles = [list(reversed(range(1, self.num_disks + 1))), [], []]  # Disks ordered largest to smallest
        self.selected_disk = None
        self.best_time = None
        self.start_time = None
        self.time_limit = None
        self.remaining_time = None
        self.font = pygame.font.Font("Grand9k Pixel.ttf", 24)  # Increased font size
        self.move_count = 0  # Track the number of moves made by the player
        self.paused = False  # Track if the game is paused

        # Initialize menu
        self.menu_loop()

    def render_text(self, text, size, color, position):
        font = pygame.font.Font("Grand9k Pixel.ttf", size)
        text_surface = font.render(text, True, color)
        text_rect = text_surface.get_rect(center=position)
        self.screen.blit(text_surface, text_rect)

    def render_button(self, text, position):
        button_rect = pygame.Rect(0, 0, 200, 50)
        button_rect.center = position
        mouse_pos = pygame.mouse.get_pos()
        if button_rect.collidepoint(mouse_pos):
            pygame.draw.rect(self.screen, (111, 143, 175), button_rect)  # White background on hover
        else:
            pygame.draw.rect(self.screen, (15, 82, 186), button_rect)  # Black background
        pygame.draw.rect(self.screen, (255, 255, 255), button_rect, 2)  # White border
        self.render_text(text, 24, self.text_color, button_rect.center)
        return button_rect
    
    def exit_game(self):
        self.destroy()  # Close the current menu window (optional)
        from MainMenu import MainMenu
        app = MainMenu()
        app.mainloop()

    def menu_loop(self):
        running = True
        while running:
            # Draw background
            self.screen.blit(self.background, (0, 0))
            
            self.render_text("Towers of Hanoi", 72, self.text_color, (self.width // 2, 100))

            play_button = self.render_button("Play", (self.width // 2, 250))
            help_button = self.render_button("Help", (self.width // 2, 350))
            exit_button = self.render_button("Exit", (self.width // 2, 450))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if play_button.collidepoint(event.pos):
                        self.play_mode_selection()
                    elif help_button.collidepoint(event.pos):
                        self.help_menu()
                    elif exit_button.collidepoint(event.pos):
                        pygame.quit()
                        sys.exit()
                        

    def play_mode_selection(self):
        running = True
        while running:
            # Draw background
            self.screen.blit(self.background, (0, 0))
            
            self.render_text("Select Game Mode", 48, self.text_color, (self.width // 2, 100))

            time_limited_button = self.render_button("Time-Limited", (self.width // 2, 250))
            endless_button = self.render_button("Endless", (self.width // 2, 350))
            back_button = self.render_button("Back", (self.width // 2, 450))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if time_limited_button.collidepoint(event.pos):
                        self.time_limit_selection()
                        return
                    elif endless_button.collidepoint(event.pos):
                        self.start_game(time_limited=False)
                        return
                    elif back_button.collidepoint(event.pos):
                        running = False

    def time_limit_selection(self):
        running = True
        while running:
            # Draw background
            self.screen.blit(self.background, (0, 0))
            
            self.render_text("Select Time Limit", 48, self.text_color, (self.width // 2, 100))

            three_min_button = self.render_button("3 Minutes", (self.width // 2, 250))
            five_min_button = self.render_button("5 Minutes", (self.width // 2, 350))
            back_button = self.render_button("Back", (self.width // 2, 450))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if three_min_button.collidepoint(event.pos):
                        self.start_game(time_limited=True, time_limit=180)
                        return
                    elif five_min_button.collidepoint(event.pos):
                        self.start_game(time_limited=True, time_limit=300)
                        return
                    elif back_button.collidepoint(event.pos):
                        running = False

    def help_menu(self):
        running = True
        while running:
            # Draw background
            self.screen.blit(self.background, (0, 0))
            
            self.render_text("Help", 48, self.text_color, (self.width // 2, 100))

            instructions = [
                "1. Move all the disks to the second or third pole.",
                "2. Only one disk can be moved at a time.",
                "3. You cannot place a larger disk on a smaller disk.",
                "4. Drag and drop disks with the mouse.",
            ]
            for i, line in enumerate(instructions):
                self.render_text(line, 24, self.text_color, (self.width // 2, 200 + i * 40))

            back_button = self.render_button("Back", (self.width // 2, 500))
            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if back_button.collidepoint(event.pos):
                        running = False

    def start_game(self, time_limited, time_limit=None):
        self.poles = [list(reversed(range(1, self.num_disks + 1))), [], []]
        self.start_time = time.time()
        self.time_limited = time_limited
        self.time_limit = time_limit
        self.remaining_time = time_limit if time_limited else None
        self.selected_disk = None
        self.move_count = 0  # Reset move count at the start of the game
        self.paused = False  # Reset pause state
        self.game_loop()

    def game_loop(self):
        running = True
        while running:
            if self.paused:
                return  # Ensure we return to the game after unpausing

            # Draw background
            self.screen.blit(self.background, (0, 0))

            # Draw poles and disks
            for i, pole in enumerate(self.poles):
                pole_x = (i + 1) * self.width // 4
                pole_y = self.height // 2 + 150  # Top of the pole
                pygame.draw.rect(self.screen, self.pole_color, (pole_x - 5, pole_y - 300, 10, 300))

                for j, disk in enumerate(pole):
                    disk_width = 30 + disk * 20
                    disk_height = 20
                    disk_x = pole_x - disk_width // 2
                    disk_y = pole_y - (j + 1) * disk_height
                    pygame.draw.rect(self.screen, self.disk_colors[disk - 1], (disk_x, disk_y, disk_width, disk_height))

                    # Add disk number
                    disk_text = str(disk)
                    self.render_text(disk_text, 18, (255, 255, 255), (disk_x + disk_width // 2, disk_y + disk_height // 2))

            # Draw selected disk
            if self.selected_disk is not None:
                disk, _ = self.selected_disk[1]  # Extract disk value
                disk_width = 30 + disk * 20
                disk_height = 20
                mouse_x, mouse_y = pygame.mouse.get_pos()
                pygame.draw.rect(self.screen, self.disk_colors[disk - 1], 
                               (mouse_x - disk_width // 2, mouse_y - disk_height // 2, disk_width, disk_height))

                # Add disk number to selected disk
                self.render_text(str(disk), 18, (255, 255, 255), (mouse_x, mouse_y))

            # Update timer
            if self.time_limited:
                self.remaining_time = self.time_limit - int(time.time() - self.start_time)
                if self.remaining_time <= 0:
                    self.time_up_popup()
                    return

                minutes = self.remaining_time // 60
                seconds = self.remaining_time % 60
                timer_text = f"Time: {minutes}:{seconds:02d}"
            else:
                elapsed_time = int(time.time() - self.start_time)
                minutes = elapsed_time // 60
                seconds = elapsed_time % 60
                timer_text = f"Time: {minutes}:{seconds:02d}"

            self.render_text(timer_text, 24, self.text_color, (self.width // 2, 50))

            # Display move count and optimal moves
            moves_text = f"Moves: {self.move_count}"
            optimal_moves_text = "Optimal Moves: 31"
            self.render_text(moves_text, 24, self.text_color, (self.width // 2, 80))
            self.render_text(optimal_moves_text, 24, self.text_color, (self.width // 2, 110))

            # Back button
            back_button = self.render_button("Pause", (self.width - 100, 50))
            randomize_button = self.render_button("Randomize", (self.width - 595, 500))
            sequence_button = self.render_button("Sequence", (self.width - 200, 500))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    mouse_x, mouse_y = event.pos

                    # Check if selecting a disk
                    if self.selected_disk is None:
                        for i, pole in enumerate(self.poles):
                            if pole:  # Ensure the pole is not empty
                                disk = pole[-1]  # Only the top disk is accessible
                                pole_x = (i + 1) * self.width // 4
                                disk_width = 30 + disk * 20
                                disk_height = 20
                                disk_x = pole_x - disk_width // 2
                                disk_y = self.height // 2 + 150 - len(pole) * disk_height

                                if disk_x <= mouse_x <= disk_x + disk_width and disk_y <= mouse_y <= disk_y + disk_height:
                                    self.selected_disk = (i, (disk, len(pole) - 1))
                                    pole.pop()
                                    break
                    else:
                        # Place disk on a pole
                        for i, pole in enumerate(self.poles):
                            pole_x = (i + 1) * self.width // 4
                            if pole_x - 50 <= mouse_x <= pole_x + 50:
                                if not pole or self.selected_disk[1][0] < pole[-1]:
                                    pole.append(self.selected_disk[1][0])
                                    self.move_count += 1  # Increment move count
                                    self.selected_disk = None
                                else:
                                    # Invalid move, return disk to original position
                                    origin_pole = self.selected_disk[0]
                                    self.poles[origin_pole].append(self.selected_disk[1][0])
                                    self.selected_disk = None
                                break

                    # Back button check
                    if back_button.collidepoint(event.pos):
                        self.paused = True
                        self.pause_menu()
                        return

                    # Randomize button check
                    if randomize_button.collidepoint(event.pos):
                        self.randomize_disks()

                    # Sequence button check
                    if sequence_button.collidepoint(event.pos):
                        self.sequence_disks()

            # Check if player won and stop the game
            if self.check_win():
                self.end_game(minutes, seconds)
                running = False  # Ensure game loop terminates

    def randomize_disks(self):
        all_disks = [disk for pole in self.poles for disk in pole]
        random.shuffle(all_disks)
        self.poles = [all_disks, [], []]

    def sequence_disks(self):
        self.poles = [list(reversed(range(1, self.num_disks + 1))), [], []]

    def check_win(self):
        # Check if all disks are arranged in the order 5, 4, 3, 2, 1 on the second or third pole
        winning_order = [5, 4, 3, 2, 1]
        for pole in self.poles[1:]:
            if pole == winning_order:
                return True
        return False

    def end_game(self, minutes, seconds):
        if self.best_time is None or (minutes * 60 + seconds) < self.best_time:
            self.best_time = minutes * 60 + seconds

        running = True
        while running:
            self.screen.fill(self.bg_color)
            self.render_text("Congratulations!", 48, self.text_color, (self.width // 2, 100))

            result_text = f"Time: {minutes}:{seconds:02d}"
            best_time_minutes = self.best_time // 60
            best_time_seconds = self.best_time % 60
            best_time_text = f"Best Time: {best_time_minutes}:{best_time_seconds:02d}"
            optimal_moves_text = "Optimal Moves: 31"
            moves_text = f"Your Moves: {self.move_count}"
            self.render_text(result_text, 36, self.text_color, (self.width // 2, 200))
            self.render_text(best_time_text, 36, self.text_color, (self.width // 2, 250))
            self.render_text(moves_text, 36, self.text_color, (self.width // 2, 300))
            self.render_text(optimal_moves_text, 36, self.text_color, (self.width // 2, 350))

            back_button = self.render_button("Back to Menu", (self.width // 2, 450))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if back_button.collidepoint(event.pos):
                        running = False

    def time_up_popup(self):
        running = True
        while running:
            self.screen.fill(self.bg_color)
            self.render_text("Time's up!", 72, (255, 0, 0), (self.width // 2, 100))

            back_button = self.render_button("Back to Menu", (self.width // 2, 450))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if back_button.collidepoint(event.pos):
                        running = False

    def pause_menu(self):
        running = True
        while running:
            self.screen.fill(self.bg_color)
            self.render_text("Pause Menu", 48, self.text_color, (self.width // 2, 100))

            return_button = self.render_button("Resume", (self.width // 2, 250))
            reset_button = self.render_button("Reset Game", (self.width // 2, 350))
            menu_button = self.render_button("Back to Menu", (self.width // 2, 450))

            pygame.display.flip()
        
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    if return_button.collidepoint(event.pos):
                        self.paused = False 
                        running = False
                        return self.game_loop()
                    elif reset_button.collidepoint(event.pos):
                        self.start_game(self.time_limited, self.time_limit)
                        return
                    elif menu_button.collidepoint(event.pos):
                        self.menu_loop()
                        return
        
if __name__ == "__main__":
    app = TowersOfHanoiGame()
    app.mainloop()
