```
C: OS/lab3/src/main.c
include <stdio.h>

#include <math.h>

#include <pthread.h>

#include <stdlib.h>

#include <time.h>



int R;

int *N;

typedef struct arguments {

    int points;

    int i;

} Arg;



double get_rand() { // возврат рандомного числа от 0 до 1

    return ((double) rand()) / RAND_MAX;

}



double get_rand_range(double min, double max) { // возвращает рандомное число от min до max

    return get_rand() * (max - min) + min;

}



void *thread_function(void *args) { // cсоздаёт n рандомных точек в квадрате размером 2*R и проверяет находятся ли точки в круге

    Arg *arg = (Arg *) args;

    int n = arg->points;

    int i = arg->i;

    for (int j = 0; j < n; j++) {

        double x = get_rand_range(-R, R);

        double y = get_rand_range(-R, R);

        if (x * x + y * y <= R * R) {

            N[i]++;

        }

    }

    return NULL;

}



int main(int argc, char *argv[]) {

    if (argc != 4) {

        printf("./a.out Radius Number_of_points Number_of_threads\n");

        exit(1);

    }

    

    R = atoi(argv[1]);

    int points_num = atoi(argv[2]), threads_num = atoi(argv[3]);

    N = (int *) calloc(threads_num, sizeof(int)); // массив точек в круге

    double time_spent = 0.0;

    clock_t begin = clock();

    pthread_t *threads = (pthread_t *) calloc(threads_num, sizeof(pthread_t));

    if (threads == NULL) {

        printf("Can't allocate memory for threads\n");

        exit(1);

    }

    int points_for_thread = points_num / threads_num;

    Arg a;

    for (int i = 0; i < threads_num; i++) {

        a.points = points_for_thread + (i < (points_num % threads_num));

        a.i = i;

        if (pthread_create(&threads[i], NULL, thread_function, &a) != 0) {

            printf("Can not create thread\n");

            exit(1);

        }



    }

    for (int i = 0; i < threads_num; i++) {

        if (pthread_join(threads[i], NULL) != 0) {

            printf("Join error\n");

            exit(1);

        }



    }

    double n = 0;

    for (int i = 0; i < threads_num; i++) { // подсчёт точек в круге

        n += N[i] / 1.0 / points_num;

    }

    printf("Monte-Carlo Circle square is %.5f\n",

           (double) 4 * R * R * n); 

    printf("Real Circle square is %.5f\n", (double) M_PI * R * R);

    clock_t end = clock();

    time_spent += (double)(end - begin);

    printf("The elapsed time is %f seconds\n", time_spent),

    free(threads);

    return 0;

}
```
