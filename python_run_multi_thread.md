在 Python 中，可以使用多种方法来编写并行程序：

使用 Python 的多线程模块 threading，可以创建多个线程来并行执行任务。例如：
import threading
 
def my_function(arg):
    # Do something with 'arg'
    print(arg)
 
threads = []
for i in range(5):
    thread = threading.Thread(target=my_function, args=(i,))
    thread.start()
    threads.append(thread)
 
# Wait for all threads to complete
for thread in threads:
    thread.join()
使用 Python 的多进程模块 multiprocessing，您可以创建多个进程来并行执行任务。例如：
import multiprocessing
 
def my_function(arg):
    # Do something with 'arg'
    print(arg)
 
processes = []
for i in range(5):
    process = multiprocessing.Process(target=my_function, args=(i,))
    process.start()
    processes.append(process)
 
# Wait for all processes to complete
for process in processes:
    process.join()

使用 Python 的线程池模块 concurrent.futures，您可以使用线程池或进程池来分配任务并行执行。例如：
import concurrent.futures
 
def my_function(arg):
    # Do something with 'arg'
    return arg
 
# Create a thread pool
with concurrent.futures.ThreadPoolExecutor() as executor:
    # Submit work to the pool
    results = [executor.submit(my_function, i) for i in range(5)]
 
    # Wait for the results to complete
    for result in concurrent.futures.as_completed(results):
        print(result.result())
 
# You can also use a process pool by replacing 'ThreadPoolExecutor' with 'ProcessPoolExecutor'

注意，在 Python 中，线程和进程的效率通常不如 C 程序中的线程和进程，因为 Python 的解释器有一个
