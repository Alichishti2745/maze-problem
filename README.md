import subprocess

def install_pyamaze():
    try: 
        subprocess.check_call(["pip", "install", "--upgrade pip"])
        subprocess.check_call(["pip", "install", "pyamaze"])
        print("Pygame installed successfully.")
    except subprocess.CalledProcessError as e:
        print(f"Error during Pygame installation: {e}")

# Check if Pygame is installed
try:
    import pyamaze
except ImportError:
    install_pyamaze()

import random
from pyamaze import maze, agent

column = 10
rows = 10
m = maze(rows, column)
m.CreateMaze(loopPercent=70)
a = agent(m, filled=True, footprints=True, shape="arrow")

direction = m.maze_map

pop_size = 400

population = []

path = []
no_of_obstacles = []
no_of_steps = []
no_of_turn = []

final_obstacle = []
final_turn = []
final_steps = []
final_fitness = []

def generatepop():
    for i in range(pop_size):
        popu = [1] + [random.randint(1, rows) for _ in range(column-2)] + [rows]
        population.append(popu)

def mutation():
    for i in population:
        i[random.randint(1, column-2)] = random.randint(1, rows)

def cross_over():
    for i in range(0, pop_size//2, 2):
        cutpoint = random.randint(1, column-2)  # Corrected cutpoint range
        parent1 = population[i]
        parent2 = population[i+1]
        child1 = parent1[:cutpoint] + parent2[cutpoint:]
        child2 = parent2[:cutpoint] + parent1[cutpoint:]
        population[(pop_size//2)+i] = child1
        population[(pop_size//2)+(i+1)] = child2

def fitness():
    # paths
    turns()
    for i in population:
        p = []
        for j in range(column-1):
            if i[j+1] - i[j] >= 0:
                for k in range(i[j], i[j+1]+1):
                    p.append((k, j+1))
            if i[j+1] - i[j] < 0:
                for k in range(i[j], i[j+1]-1, -1):
                    p.append((k, j+1))
        p.append((rows, column))
        path.append(p)

    # obstacles
    obs = 0
    for i in path:
        for j in range(len(i)-1):
            if i[j+1][0]-i[j][0] >= 0 and i[j+1][1] == i[j][1]:
                if direction[i[j]]["S"] == 0:
                    obs += 1
            if i[j+1][0]-i[j][0] < 0 and i[j+1][1] == i[j][1]:
                if direction[i[j]]["N"] == 0:
                    obs += 1
            if i[j+1][1]-i[j][1] >= 0 and i[j+1][0] == i[j][0]:
                if direction[i[j]]["E"] == 0:
                    obs += 1
            if i[j+1][1]-i[j][1] < 0 and i[j+1][0] == i[j][0]:
                if direction[i[j]]["W"] == 0:
                    obs += 1
        no_of_obstacles.append(obs)
        obs = 0

    # no of steps
    for i in path:
        no_of_steps.append(len(i))

    # formulas
    max_obstacles = max(no_of_obstacles)
    min_obstacles = min(no_of_obstacles)
    max_turn = max(no_of_turn)
    min_turn = min(no_of_turn)
    max_step = max(no_of_steps)
    min_step = min(no_of_steps)

    w_obs = 3
    w_turn = 2
    w_path = 2

    for i in range(pop_size):
        f_obs = 1 - ((no_of_obstacles[i]-min_obstacles)/(max_obstacles-min_obstacles))
        final_obstacle.append(f_obs)
        f_turn = 1 - ((no_of_turn[i]-min_turn)/(max_turn-min_turn))
        final_turn.append(f_turn)
        f_steps = 1 - ((no_of_steps[i]-min_step)/(max_step-min_step))
        final_steps.append(f_steps)
        final_fitness.append((100*w_obs*f_obs) * (((w_path * f_steps) + (w_turn * f_turn)) / (w_path + w_turn)))

def turns():
    for i in population:
        turn = 0
        for j in range(0, column-2):
            if i[j] != i[j+1]:
                turn += 1
        no_of_turn.append(turn)

def sorting():
    for i in range(pop_size - 1):
        for j in range(i+1, pop_size):
            if final_fitness[j] > final_fitness[i]:
                population[i], population[j] = population[j], population[i]
                final_fitness[i], final_fitness[j] = final_fitness[j], final_fitness[i]


dictionary = {}

def solution():
    sol = []
    for i in range(pop_size):
        if final_fitness[i] > 0 and no_of_obstacles[i] == 0:
            sol = path[i]
            for j in range(len(sol)-1):
                dictionary.update({sol[j+1]: sol[j]})
            return 1
    return 0

# Main Function

generatepop()
i = 0
while True:
    i += 1
    fitness()
    if solution():
        print(f"Solution found in iteration = {i}")
        m.tracePath({a: dictionary})
        m.run()
        break

    sorting()
    cross_over()
    mutation()

    path = []
    no_of_obstacles = []
    no_of_steps = []
    no_of_turn = []

    final_obstacle = []
    final_turn = []
    final_steps = []
    final_fitness = []
