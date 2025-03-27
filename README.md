# SoLong

这个 **So_Long 项目** 旨在通过实现一个基础的 2D 游戏，帮助你掌握以下 **核心技能** 和 **图形编程知识**：

---

### **核心学习目标**
1. **图形库与窗口管理**
- 使用 **MiniLibX** 库创建窗口、加载纹理（textures）和精灵（sprites），掌握基本的图形渲染流程。
- 处理窗口事件（如关闭窗口、键盘输入），实现用户交互逻辑。

2. **地图解析与验证**
- 解析 `.ber` 格式的地图文件，验证其合法性：
- 必须被墙壁（`1`）包围。
- 包含唯一的玩家起始点（`P`）和出口（`E`）。
- 至少有一个可收集物（`C`），且所有元素路径可达（通过 BFS/DFS 算法验证）。
- 处理非法输入（如重复字符、非矩形地图），输出明确的错误信息。

3. **游戏逻辑与状态管理**
- 实现玩家移动（WASD/方向键），检测碰撞（墙壁阻挡、可收集物拾取）。
- 追踪游戏状态：收集物数量、移动步数、出口条件（需收集全部物品）。
- 在终端或屏幕上实时显示移动步数（Bonus 要求屏幕显示）。

4. **内存管理与健壮性**
- 动态分配内存存储地图数据，确保无泄漏（`free`）。
- 处理图形资源（如纹理图片）的加载与释放，避免资源泄漏。

5. **错误处理与防御式编程**
- 检查文件读取错误（如文件不存在、权限不足）、地图格式错误。
- 确保程序在异常情况下（如地图验证失败）能优雅退出，释放所有资源。

---

### **关键技术点解析**
#### 1. MiniLibX 基础使用
```c
	// 初始化 MiniLibX
	void *mlx = mlx_init();
	void *win = mlx_new_window(mlx, width, height, "So_Long");

	// 加载纹理
	void *img = mlx_xpm_file_to_image(mlx, "texture.xpm", &width, &height);

	// 渲染纹理到窗口
	mlx_put_image_to_window(mlx, win, img, x * TILE_SIZE, y * TILE_SIZE);

	// 注册键盘事件
	mlx_hook(win, KeyPress, KeyPressMask, handle_keypress, NULL);

	// 主事件循环
	mlx_loop(mlx);
```
**关键点**：正确处理图像资源生命周期，避免窗口和图像未释放。

#### 2. 地图路径验证（BFS 示例）
```c
	bool	is_path_valid(t_map *map)
	{
		// 复制地图，标记可达区域
		int **visited = init_visited(map);
		Queue *q = create_queue();
		enqueue(q, map->player_pos);
		visited[map->player_pos.y][map->player_pos.x] = 1;

		while (!is_empty(q))
		{
			Point p = dequeue(q);
			// 检查相邻四方向
			for (int i = 0; i < 4; i++)
			{
				Point next = {p.x + dx[i], p.y + dy[i]};
				if (is_valid(next, map) && !visited[next.y][next.x])
				{
					if (map->grid[next.y][next.x] == 'E' || map->grid[next.y][next.x] == 'C')
						collectibles--;
					visited[next.y][next.x] = 1;
					enqueue(q, next);
				}
			}
		}
		// 检查是否所有 C 和 E 可达
		return (collectibles == 0 && exit_reachable);
	}
```
**关键点**：使用广度优先搜索（BFS）确保玩家能到达所有 `C` 和 `E`。

#### 3. 游戏状态更新
```c
	void	move_player(t_game *game, int dx, int dy)
	{
		int new_x = game->player_x + dx;
		int new_y = game->player_y + dy;

		if (game->map[new_y][new_x] == '1')
			return ; // 碰撞检测
		if (game->map[new_y][new_x] == 'C')
			game->collected++;
		if (game->map[new_y][new_x] == 'E' && game->collected == game->total_collectibles)
		{
			printf("You Win!\n");
			exit(0);
		}
		game->player_x = new_x;
		game->player_y = new_y;
		game->steps++;
		printf("Steps: %d\n", game->steps); // Bonus 可改为屏幕渲染
	}
```
**关键点**：实时更新玩家位置和状态，检测胜利条件。

---

### **工程实践与思维提升**
1. **模块化设计**
- 分离图形渲染、地图解析、游戏逻辑到不同文件（如 `graphics.c`、`map_parser.c`、`game_logic.c`）。
- 使用头文件（如 `so_long.h`）定义共享数据结构和常量。

2. **防御式编程**
- 校验地图文件的每个字符（仅允许 `0, 1, C, E, P`）。
- 处理 MiniLibX 初始化失败（如 `mlx_init()` 返回 `NULL`）。

3. **性能优化**
- 使用静态内存分配优化小地图，避免频繁 `malloc`。
- 预加载所有纹理，减少运行时文件 I/O 开销。

4. **测试策略**
- 设计多种地图测试用例（合法/非法、不同尺寸、复杂路径）。
- 使用脚本自动化测试（如遍历移动、收集物品、触发胜利/失败条件）。

---

### **项目隐喻与深层价值**
- **“游戏是系统编程的综合体现”**：通过实现游戏逻辑，融合文件操作、内存管理、事件处理和图形渲染，理解复杂系统的构建。
- **“用户体验与底层细节的平衡”**：在实现功能的同时，优化交互反馈（如平滑移动、即时状态更新），培养全栈思维。
- **“从需求到实现”**：将游戏设计文档转化为可执行代码，强化工程化开发能力。

---

### **通关要点**
1. **分步实现**
- 先完成地图解析和基础渲染，再添加玩家移动和碰撞检测。
- 最后实现路径验证和胜利条件，逐步集成测试。

2. **错误处理全覆盖**
- 处理文件读取错误（如 `perror("Error")`）、地图验证失败、图形初始化失败。
- 确保所有错误路径释放资源，避免内存泄漏。

3. **扩展 Bonus 功能**
- 添加敌人 AI（简单移动逻辑），检测碰撞触发失败。
- 实现精灵动画（多帧纹理切换），使用 `mlx_loop_hook` 控制帧率。
- 在窗口内渲染文本（如步数计数器），替代终端输出。

---

通过此项目，你将从“控制台开发者”进阶为“图形应用开发者”，掌握游戏开发的核心技术栈，为更复杂的图形项目（如 3D 渲染、游戏引擎）奠定基础。
