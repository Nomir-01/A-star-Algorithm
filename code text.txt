import pygame
from queue import PriorityQueue

class Pygame:
    def __init__(self, row, column, width, total_rows):
        self.row = row
        self.col = column
        self.x = row * width
        self.y = column * width
        self.color = Black
        self.width = width
        self.total_rows = total_rows

    def get_pos(self):
        return self.row, self.col

    def select_start(self):
        self.color = Orange

    def select_end(self):
        self.color = Red

    def select_obstacle(self):
        self.color = Blue

    def select_reset(self):
        self.color = Black

    def draw(self, window):
        pygame.draw.rect(window, self.color, (self.x, self.y, self.width, self.width))

    def update_neighbors(self, grid):
        self.neighbors = []
        if self.row < self.total_rows - 1 and not grid[self.row + 1][self.col].obstacle():
            self.neighbors.append(grid[self.row + 1][self.col])

        if self.row > 0 and not grid[self.row - 1][self.col].obstacle():
            self.neighbors.append(grid[self.row - 1][self.col])

        if self.col < self.total_rows - 1 and not grid[self.row][self.col + 1].obstacle():
            self.neighbors.append(grid[self.row][self.col + 1])

        if self.col > 0 and not grid[self.row][self.col - 1].obstacle():
            self.neighbors.append(grid[self.row][self.col - 1])

    def obstacle(self):
        return self.color == Blue

    def open_path(self):
        self.color = Green

    def closed_path(self):
        self.color = Yellow

    def final_path(self):
        self.color = Gray


class ShortestPath:
    def __init__(self):
        self.width = 660
        self.window = pygame.display.set_mode((self.width, self.width))
        pygame.display.set_caption('Shortest Path')

    @staticmethod
    def layout(rows, width):
        grid = []
        gap = width // rows
        for i in range(rows):
            grid.append([])
            for j in range(rows):
                pg_window = Pygame(i, j, gap, rows)
                grid[i].append(pg_window)
        return grid

    @staticmethod
    def lines(window, rows, width):
        gap = width // rows
        for i in range(rows):
            pygame.draw.line(window, White, (0, i * gap), (width, i * gap), 2)
            for j in range(rows):
                pygame.draw.line(window, White, (j * gap, 0), (j * gap, width), 2)

    @staticmethod
    def draw(window, grid, rows, width):
        for row in grid:
            for pg_window in row:
                pg_window.draw(window)
        ShortestPath.lines(window, rows, width)
        pygame.display.update()

    @staticmethod
    def select_box(pos, rows, width):
        gap = width // rows
        y, x = pos
        row = y // gap
        col = x // gap
        return row, col

    @staticmethod
    def algorithm(draw, grid, start, end):
        counter = 0
        new_set = PriorityQueue()
        new_set.put((0, counter, start))
        prev_node = {}
        value_g = {spot: float('inf') for row in grid for spot in row}
        value_g[start] = 0
        value_f = {spot: float('inf') for row in grid for spot in row}
        value_f[start] = ShortestPath.value_h(start.get_pos(), end.get_pos())
        new_set_hash = {start}
        while not new_set.empty():
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()

            current_node = new_set.get()[2]
            new_set_hash.remove(current_node)

            if current_node == end:
                ShortestPath.new_path(prev_node, end, draw)
                end.final_path()
                return True

            for neighbor in current_node.neighbors:
                Tvalue_g = value_g[current_node] + 1

                if Tvalue_g < value_g[neighbor]:
                    prev_node[neighbor] = current_node
                    value_g[neighbor] = Tvalue_g
                    value_f[neighbor] = Tvalue_g + ShortestPath.value_h(neighbor.get_pos(), end.get_pos())
                    if neighbor not in new_set_hash:
                        counter += 1
                        new_set.put((value_f[neighbor], counter, neighbor))
                        new_set_hash.add(neighbor)
                        neighbor.open_path()
            draw()
            if current_node != start:
                current_node.closed_path()

        return False

    @staticmethod
    def value_h(a1, a2):
        x1, y1 = a1
        x2, y2 = a2
        return abs(x1 - x2) + abs(y1 - y2)

    @staticmethod
    def new_path(prev_node, current_node, draw):
        while current_node in prev_node:
            current_node = prev_node[current_node]
            current_node.final_path()
            draw()

Black = (20, 20, 20)
White = (200, 200, 200)
Blue = (48, 152, 196)
Red = (229, 120, 108)
Orange = (249, 150, 67)
Green = (54, 208, 141)
Yellow = (242, 219, 70)
Gray = (188, 181, 174)

SP = ShortestPath()
line_gap = 15
grid = ShortestPath.layout(line_gap, SP.width)
start = None
end = None
run = True
while run:
    SP.draw(SP.window, grid, line_gap, SP.width)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            run = False

        if pygame.mouse.get_pressed()[0]:
            pos = pygame.mouse.get_pos()
            row, col = SP.select_box(pos, line_gap, SP.width)
            spot = grid[row][col]
            if not start and spot != end:
                start = spot
                start.select_start()

            elif not end and spot != start:
                end = spot
                end.select_end()

            elif spot != end and spot != start:
                spot.select_obstacle()

        elif pygame.mouse.get_pressed()[2]:
            pos = pygame.mouse.get_pos()
            row, col = SP.select_box(pos, line_gap, SP.width)
            spot = grid[row][col]
            spot.select_reset()
            if spot == start:
                start = None
            elif spot == end:
                end = None

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_RETURN and start and end:
                for row in grid:
                    for spot in row:
                        spot.update_neighbors(grid)
                SP.algorithm(lambda: SP.draw(SP.window,grid,line_gap,SP.width), grid, start, end)

            if event.key == pygame.K_r:
                start = None
                end = None
                grid = SP.layout(line_gap, SP.width)

pygame.quit()