
---
layout: post
title: "借助chatgpt实现一个游戏玩法"
subtitle: '一个消除类游戏的demo'
author: "chaoxiaodi"
header-style: text
tags:
  - game
  - 消除类游戏
  - pygame
  - match game
  - chatgpt
---

### 前言
之前写过一个消除类游戏玩法的设计

### 蹭个chatgpt的热度

现在借助AI写代码 那真是 爽爆了

只要你能描述清楚 生成的代码基本就能直接跑 比自己写的好多了

Talk is cheap. Show me the code.

话不多说 直接上代码

    import pygame
    import sys
    import random
    import numpy as np

    # 初始化 Pygame
    pygame.init()

    # 设置常量
    CELL_SIZE = 40          # 单元格大小
    GRID_WIDTH = 12         # 横向单元格数量
    GRID_HEIGHT = 16        # 纵向单元格数量
    PREVIEW_CELL_SIZE = 12  # 预览区域的格子大小
    TOP_MARGIN = 100        # 顶部边距
    WINDOW_WIDTH = CELL_SIZE * GRID_WIDTH + 1 # +1 让最右侧能够显示边框线
    WINDOW_HEIGHT = CELL_SIZE * GRID_HEIGHT + TOP_MARGIN + 1 # +1 让最下面能够显示边框线

    # 颜色定义
    BLACK = (0, 0, 0)
    WHITE = (255, 255, 255)
    GRAY = (128, 128, 128)
    COLORS = [
        (255, 0, 0),    # 红
        (0, 255, 0),    # 绿
        (0, 0, 255),    # 蓝
        (255, 255, 0),  # 黄
        (255, 0, 255),  # 紫
        (0, 255, 255),  # 青
    ]

    # 目标形状定义
    # 形状参考俄罗斯方块的7种形状
    TARGET_SHAPES = {
        'I': np.array([[1],
                      [1],
                      [1],
                      [1]]),
        'L': np.array([[1, 0],
                      [1, 0],
                      [1, 1]]),
        'J': np.array([[0, 1],
                      [0, 1],
                      [1, 1]]),
        'O': np.array([[1, 1],
                      [1, 1]]),
        'Z': np.array([[1, 1, 0],
                      [0, 1, 1]]),
        'S': np.array([[0, 1, 1],
                      [1, 1, 0]]),
        'T': np.array([[1, 1, 1],
                      [0, 1, 0]])
    }

    # 游戏关卡配置
    LEVEL_CONFIG= {
        "1": {
            'target_sharp_numer': 3,
            'target_score': 3
        },
        "2": {
            'target_sharp_numer': 3,
            'target_score': 10
        },
        "3": {
            'target_sharp_numer': 3,
            'target_score': 30
        }
    }

    # 创建游戏窗口
    screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("拼图消除游戏")


    class TargetShape:
        def __init__(self):
            self.shape = random.choice(list(TARGET_SHAPES.values()))
            self.color = random.choice(COLORS)
            self.random_rotate()

        def random_rotate(self):
            """通过随机旋转把7种形状扩展为更多形状"""
            self.shape = np.rot90(self.shape, k=random.choice([0, 1, 2, 3]))


    class Block:
        def __init__(self):
            self.x = GRID_WIDTH // 2 - 1
            self.y = 0
            self.shape = [[1]]
            self.color = random.choice(COLORS)

        def move_down(self):
            self.y += 1

        def move_left(self):
            if self.x > 0:
                self.x -= 1

        def move_right(self):
            if self.x < GRID_WIDTH - len(self.shape[0]):
                self.x += 1


    class Game:
        def __init__(self):
            self.grid = self.create_grid()
            self.colors_grid = self.create_colors_grid()
            self.score = 0
            self.level = 1
            self.font = pygame.font.Font(None, 36)
            self.target_shapes = [TargetShape() for _ in range(3)]
            self.last_positions = []

        def create_grid(self):
            return [[0 for _ in range(GRID_WIDTH)] for _ in range(GRID_HEIGHT)]

        def create_colors_grid(self):
            return [[None for _ in range(GRID_WIDTH)] for _ in range(GRID_HEIGHT)]

        def check_collision(self, block):
            '''判断方块是否可以继续下落
                判断方块是否可以左右移动
                判断方块是否可以旋转'''
            for y, row in enumerate(block.shape):
                for x, cell in enumerate(row):
                    if cell:
                        if block.y + y >= GRID_HEIGHT:
                            return True
                        if (block.y + y >= 0 and 
                            self.grid[block.y + y][block.x + x]):
                            return True
            return False

        def merge_block_to_grid(self, block):
            '''把位置和颜色填充到grid 并更新方块最后的位置'''
            '''enumerate 同时获取 index 和 元素'''
            for y, row in enumerate(block.shape):
                for x, cell in enumerate(row):
                    if cell:
                        self.grid[block.y + y][block.x + x] = 1
                        self.colors_grid[block.y + y][block.x + x] = block.color
                        self.last_positions = [(block.y + y, block.x + x)]

        def check_shapes_at_positions(self):
            """只检查包含指定位置的可能形状"""
            for target_shape in self.target_shapes:
                shape_height, shape_width = target_shape.shape.shape
                
                # 对每个新落下的方块位置进行检查
                for pos_y, pos_x in self.last_positions:
                    # 计算需要检查的区域范围
                    start_y = max(0, pos_y - shape_height + 1)
                    end_y = min(GRID_HEIGHT - shape_height + 1, pos_y + 1)
                    start_x = max(0, pos_x - shape_width + 1)
                    end_x = min(GRID_WIDTH - shape_width + 1, pos_x + 1)
                    
                    # 在这个范围内检查匹配
                    for y in range(start_y, end_y + 1):
                        for x in range(start_x, end_x + 1):
                            # 检查是否会超出网格边界
                            # 上面计算区域范围时忽略了方块在最右侧或这最左侧的情况
                            if y + shape_height > GRID_HEIGHT or x + shape_width > GRID_WIDTH:
                                continue
                            if self.check_shape_at_position(target_shape, y, x):
                                return True
            return False

        def check_shape_at_position(self, target_shape, y, x):
            """检查特定位置是否匹配目标形状"""
            shape_height, shape_width = target_shape.shape.shape
            matched_positions = set()
            color = None

            # 第一步：检查目标形状位置的匹配情况
            for i in range(shape_height):
                for j in range(shape_width):
                    if target_shape.shape[i][j] == 1:
                        if not self.grid[y+i][x+j]:
                            return False
                        if color is None:
                            color = self.colors_grid[y+i][x+j]
                        elif self.colors_grid[y+i][x+j] != color:
                            return False
                        matched_positions.add((y+i, x+j))

            # 检查颜色是否匹配目标形状
            if color != target_shape.color:
                return False

            # 第二步：检查matched_positions中的方块是否相互连接
            def are_positions_connected(positions):
                if not positions:
                    return True
                
                # 使用BFS检查连接性
                visited = set()
                start = next(iter(positions))
                queue = [start]
                visited.add(start)

                while queue:
                    curr_y, curr_x = queue.pop(0)
                    for dy, dx in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                        new_y, new_x = curr_y + dy, curr_x + dx
                        if (new_y, new_x) in positions and (new_y, new_x) not in visited:
                            visited.add((new_y, new_x))
                            queue.append((new_y, new_x))

                return len(visited) == len(positions)

            # 验证匹配位置的连接性
            if not are_positions_connected(matched_positions):
                return False

            # 如果所有检查都通过，移除匹配的方块
            for pos_y, pos_x in matched_positions:
                self.grid[pos_y][pos_x] = False
                self.colors_grid[pos_y][pos_x] = None

            # 更新目标形状
            index = self.target_shapes.index(target_shape)
            self.target_shapes[index] = TargetShape()

            # 处理掉落
            self.handle_falling_blocks()
            return True

        def handle_falling_blocks(self):
            """处理方块掉落"""
            falling = True
            while falling:
                falling = False
                for x in range(GRID_WIDTH):
                    for y in range(GRID_HEIGHT-1, 0, -1):  # 从底部向上，但不包括最顶行
                        if not self.grid[y][x] and self.grid[y-1][x]:  # 如果当前位置为空且上方有方块
                            # 移动方块
                            self.grid[y][x] = True
                            self.grid[y-1][x] = False
                            self.colors_grid[y][x] = self.colors_grid[y-1][x]
                            self.colors_grid[y-1][x] = None
                            falling = True

    def draw_grid_lines():
        """绘制网格线"""
        for x in range(0, WINDOW_WIDTH, CELL_SIZE):
            pygame.draw.line(screen, GRAY, (x, TOP_MARGIN), (x, WINDOW_HEIGHT))
        for y in range(TOP_MARGIN, WINDOW_HEIGHT, CELL_SIZE):
            pygame.draw.line(screen, GRAY, (0, y), (WINDOW_WIDTH, y))

    def draw_blocks(game):
        """绘制固定的方块"""
        for y, row in enumerate(game.grid):
            for x, cell in enumerate(row):
                if cell:
                    rect = pygame.Rect(
                        x * CELL_SIZE,
                        y * CELL_SIZE + TOP_MARGIN,
                        CELL_SIZE,
                        CELL_SIZE
                    )
                    pygame.draw.rect(screen, game.colors_grid[y][x], rect)

    def draw_current_block(block):
        """绘制当前移动的方块"""
        for y, row in enumerate(block.shape):
            for x, cell in enumerate(row):
                if cell:
                    rect = pygame.Rect(
                        (block.x + x) * CELL_SIZE,
                        (block.y + y) * CELL_SIZE + TOP_MARGIN,
                        CELL_SIZE,
                        CELL_SIZE
                    )
                    pygame.draw.rect(screen, block.color, rect)

    def draw_score(game):
        """绘制分数"""
        level_test = game.font.render(f'Level: {game.level}', True, WHITE)
        score_text = game.font.render(f'Score: {game.score}', True, WHITE)
        screen.blit(level_test, (10, 20))
        screen.blit(score_text, (10, 60))

    def draw_target_shapes(game):
        """绘制所有目标形状"""
        title = game.font.render(f'Targets:', True, WHITE)
        screen.blit(title, (150, 20))

        for i, target_shape in enumerate(game.target_shapes):
            # 为每个形状计算位置
            preview_x = 300 + i * (PREVIEW_CELL_SIZE * 4 + 10)
            preview_y = 30

            # 绘制形状
            for y, row in enumerate(target_shape.shape):
                for x, cell in enumerate(row):
                    if cell:
                        rect = pygame.Rect(
                            preview_x + x * PREVIEW_CELL_SIZE,
                            preview_y + y * PREVIEW_CELL_SIZE,
                            PREVIEW_CELL_SIZE,
                            PREVIEW_CELL_SIZE
                        )
                        pygame.draw.rect(screen, target_shape.color, rect)

    def show_game_over(score):
        """显示游戏结束界面"""
        font = pygame.font.Font(None, 48)
        small_font = pygame.font.Font(None, 36)  # 较小的字体用于提示文本
        game_over_text = font.render('Game Over', True, WHITE)
        score_text = font.render(f'Final Score: {score}', True, WHITE)
        restart_text = small_font.render('Press ENTER to Restart', True, WHITE)
        quit_text = small_font.render('Press ESC to Quit', True, WHITE)

        # 创建半透明的背景
        overlay = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
        overlay.fill(BLACK)
        overlay.set_alpha(128)
        screen.blit(overlay, (0, 0))
        
        # 显示文本
        screen.blit(game_over_text, 
                    (WINDOW_WIDTH//2 - game_over_text.get_width()//2, 
                    WINDOW_HEIGHT//2 - 60))
        screen.blit(score_text, 
                    (WINDOW_WIDTH//2 - score_text.get_width()//2, 
                    WINDOW_HEIGHT//2 - 30))
        screen.blit(restart_text,
                    (WINDOW_WIDTH//2 - restart_text.get_width()//2,
                    WINDOW_HEIGHT//2 + 30))
        screen.blit(quit_text,
                    (WINDOW_WIDTH//2 - quit_text.get_width()//2,
                    WINDOW_HEIGHT//2 + 60))

        pygame.display.flip()

        # 等待几秒或等待按键
        waiting = True
        while waiting:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_RETURN:
                        waiting = False
                    elif event.key == pygame.K_ESCAPE:
                        pygame.quit()
                        sys.exit()

    def main():
        clock = pygame.time.Clock()
        block = Block()
        game = Game()
        fall_time = 0
        fall_speed = 500

        while True:
            current_time = pygame.time.get_ticks()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()

                # 按键判断 asd 左下右 同时支持
                if event.type == pygame.KEYDOWN:
                    if event.key in [pygame.K_LEFT, pygame.K_a]:
                        block.move_left()
                        if game.check_collision(block):
                            block.x += 1
                    elif event.key in [pygame.K_RIGHT, pygame.K_d]:
                        block.move_right()
                        if game.check_collision(block):
                            block.x -= 1
                    elif event.key in [pygame.K_DOWN, pygame.K_s]:
                        block.move_down()
                        if game.check_collision(block):
                            block.y -= 1
            
            if current_time - fall_time > fall_speed:
                block.move_down()

                # 判断是否到达底部
                if game.check_collision(block):
                    block.y -= 1
                    game.merge_block_to_grid(block)

                    # 检查形状匹配并更新分数
                    if game.check_shapes_at_positions():
                        game.score += 1

                    block = Block()
                    if game.check_collision(block):
                        show_game_over(game.score)
                        game = Game()
                        block = Block()
                fall_time = current_time

            screen.fill(BLACK)
            draw_target_shapes(game)
            draw_blocks(game)
            draw_current_block(block)
            draw_grid_lines()
            draw_score(game)

            pygame.display.flip()
            clock.tick(60)

    if __name__ == '__main__':
        main()

### 一点想法

之前google 有一个 提问的艺术 说的是遇到问题时怎么去提问

现在对AI写提示词也是一种艺术

如果只是简单的单一问题 可能一句话就描述了

如果要系统的实现某些功能

需要有比较清晰的逻辑和条理

也是一种语言表达的锻炼和提升

### TODO

这个代码只实现了最基础的功能

方块下落 计算是否匹配图形 消除图形 重新生成图形 方块掉落 积分等

后续还可以增加一些功能

  - 关卡模式
  - 积分模式/挑战模式
  - 特殊道具 消除整行/消除整列/消除以自己为中心的9宫格
  - 隐藏图形 如 消除后掉落如果达成五连/九宫格等 额外加分
  - 游戏记录
  - 增加更多属性 比如记录一共用了多少方块 
  - 预制图形 从同色扩展为多色等


### 后记

pip install pygame numpy

玩法大概就是会显示3个预制的图形

移动方块拼成预制的图形 然后就回被消除并获得1分

![](/img/shapecleargame.png)


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
