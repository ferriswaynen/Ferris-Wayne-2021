import random as rand
import numpy as np
import matplotlib.pyplot as plt


class Ind:
    def __init__(self, sck=False, ctg=0, inf=0, pod=None):
        self.sck = sck  # sick/infected at some point
        self.ctg = ctg  # contagious period
        self.inf = inf  # number of people you infected
        self.pod = pod  # pod number


def rando(lst):
    return lst[rand.randint(0, len(lst)-1)]


def infect(p):
    p.sck = True
    p.ctg = 12


def gen_pop(n):
    global pop
    pop = []
    for i in range(n):
        pop.append(Ind())
    p0 = rando(pop)
    infect(p0)


def gen_pods(k):
    global pods
    pods = []
    if k:
        for i in range(0, len(pop), k):
            pods.append(pop[i:i+k])
            for p in pop[i:i+k]:
                p.pod = i//k


def contact(p, b):
    if rand.choices([0, 1], [1-b, b], k=1)[0] and pods:
        pod = pods[p.pod]
        caught = rando(pod)
    else:
        caught = rando(pop)
    timing = {1: .012, 2: .013, 3: .015, 4: .02, 5: .03, 6: .045, 7: .07, 8: .095, 9: .125, 10: .16, 11: .18, 12: .15}
    r = 2.5  # R0
    x = r/9 * timing[p.ctg]
    if rand.choices([0, 1], [1-x, x], k=1)[0] and not caught.sck:
        p.inf += 1
        infect(caught)


def run(n, s, t, m, d=0, b=1/2):  # number of sims, population size, days, interactions per day, pod size, pod interaction chance
    global avg_sck
    sim_sck = []
    for i in range(n):
        gen_pop(s)
        gen_pods(d)
        daily_sck = []
        for j in range(t+1):
            for k in range(m):
                for p in pop:
                    if p.ctg:
                        contact(p, b)
            for p in pop:
                if p.ctg:
                    p.ctg -= 1
            daily_sck.append(sum(1 for p in pop if p.sck))
        sim_sck.append(daily_sck)
    arrays = [np.array(x) for x in sim_sck]
    avg_sck = list(map(round, [np.mean(k) for k in zip(*arrays)]))
    print(f'{avg_sck[-1]}\t(reached by day {avg_sck.index(avg_sck[-1])})')
    tots = [x[-1] for x in sim_sck]
    print('Stdev:', np.std(tots, ddof=1))


def plot(ln, c):
    colors = ['b', 'g', 'r', 'c', 'm', 'y', 'k', 'tab:orange', 'tab:purple', 'tab:brown', 'tab:pink', 'tab:gray']
    plt.plot(range(len(avg_sck)), avg_sck, colors[c], label=ln)


def show():
    plt.xlabel('Days')
    plt.ylabel('Cumulative number of infections in population')
    plt.axis([0, 50, 0, 10000])
    plt.legend()
    plt.show()


def main():
    test_1 = {'No pods, 5 interactions/day': [50, 5],
              '5/pod, 5 interactions/day 50% in pod': [50, 5, 5],
              '5/pod, 5 interactions/day 75% in pod': [50, 5, 5, 3 / 4],
              '5/pod, 5 interactions/day 90% in pod': [50, 5, 5, 9 / 10],
              'No pods, 10 interactions/day': [50, 10],
              '5/pod, 10 interactions/day 50% in pod': [50, 10, 5],
              '5/pod, 10 interactions/day 75% in pod': [50, 10, 5, 3 / 4],
              '5/pod, 10 interactions/day 90% in pod': [50, 10, 5, 9 / 10],
              'No pods, 20 interactions/day': [50, 20],
              '5/pod, 20 interactions/day 50% in pod': [50, 20, 5],
              '5/pod, 20 interactions/day 75% in pod': [50, 20, 5, 3 / 4],
              '5/pod, 20 interactions/day 90% in pod': [50, 20, 5, 9 / 10]}
    test_2 = {'No pods, 5 interactions/day': [50, 5],
              '10/pod, 5 interactions/day 50% in pod': [50, 5, 10],
              '10/pod, 5 interactions/day 75% in pod': [50, 5, 10, 3 / 4],
              '10/pod, 5 interactions/day 90% in pod': [50, 5, 10, 9 / 10],
              'No pods, 10 interactions/day': [50, 10],
              '10/pod, 10 interactions/day 50% in pod': [50, 10, 10],
              '10/pod, 10 interactions/day 75% in pod': [50, 10, 10, 3 / 4],
              '10/pod, 10 interactions/day 90% in pod': [50, 10, 10, 9 / 10],
              'No pods, 20 interactions/day': [50, 20],
              '10/pod, 20 interactions/day 50% in pod': [50, 20, 10],
              '10/pod, 20 interactions/day 75% in pod': [50, 20, 10, 3 / 4],
              '10/pod, 20 interactions/day 90% in pod': [50, 20, 10, 9 / 10]}
    test_3 = {'No pods, 5 interactions/day': [50, 5],
              '20/pod, 5 interactions/day 50% in pod': [50, 5, 20],
              '20/pod, 5 interactions/day 75% in pod': [50, 5, 20, 3 / 4],
              '20/pod, 5 interactions/day 90% in pod': [50, 5, 20, 9 / 10],
              'No pods, 10 interactions/day': [50, 10],
              '20/pod, 10 interactions/day 50% in pod': [50, 10, 20],
              '20/pod, 10 interactions/day 75% in pod': [50, 10, 20, 3 / 4],
              '20/pod, 10 interactions/day 90% in pod': [50, 10, 20, 9 / 10],
              'No pods, 20 interactions/day': [50, 20],
              '20/pod, 20 interactions/day 50% in pod': [50, 20, 20],
              '20/pod, 20 interactions/day 75% in pod': [50, 20, 20, 3 / 4],
              '20/pod, 20 interactions/day 90% in pod': [50, 20, 20, 9 / 10]}
    tests = [test_1, test_2, test_3]
    for test in tests:
        for i in range(len(test)):
            print(list(test.keys())[i], end=': ')
            run(100, 10000, *list(test.values())[i])  # number of sims, population size, other test conditions
            plot(list(test.keys())[i], i)
        show()


if __name__ == '__main__':
    main()
