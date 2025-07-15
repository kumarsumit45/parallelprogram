 Question 1: Sum of Odd Numbers using Scatter and Gather


#include <mpi.h>
 #include <stdio.h>
 #include <stdlib.h>
 int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);
    
    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    
    const int N = 10; // Array size
    int *array = NULL;
    int chunk_size = N / size;
    int *local_array = (int*)malloc(chunk_size * sizeof(int));
    
    // Root process initializes the array
    if (rank == 0) {
        array = (int*)malloc(N * sizeof(int));
        for (int i = 0; i < N; i++) {
            array[i] = i + 1; // Array: 1, 2, 3, ..., 10 // rand() % 1000; // Random values 0-999
        }
	printf("Array elements :\n");
        for(int i=0;i<N;i++){
            printf("%d \t",array[i]);
        }
    }
    
    // Scatter array chunks to all processes
    MPI_Scatter(array, chunk_size, MPI_INT, 
                local_array, chunk_size, MPI_INT, 
                0, MPI_COMM_WORLD);
    
    // Each process finds sum of odd numbers in its chunk
    int local_sum = 0;
    for (int i = 0; i < chunk_size; i++) {
        if (local_array[i] % 2 == 1) { // Check if odd
            local_sum += local_array[i];
        }
    }
    
    // Gather all local sums to root process
    int *all_sums = NULL;
    if (rank == 0) {
        all_sums = (int*)malloc(size * sizeof(int));
    }
    
    MPI_Gather(&local_sum, 1, MPI_INT, 
               all_sums, 1, MPI_INT, 
               0, MPI_COMM_WORLD);
    
    // Root process calculates final sum
 if (rank == 0) {
        int total_sum = 0;
        for (int i = 0; i < size; i++) {
            total_sum += all_sums[i];
        }
        printf("Sum of all odd numbers: %d\n", total_sum);
        free(all_sums);
        free(array);
    }
    
    free(local_array);
    MPI_Finalize();
    return 0;
 }