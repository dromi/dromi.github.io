---
layout: post
title:  "Scheduling and monitoring independent concurrent processes in python"
categories: python
---

For a problem that requires many intensive tasks to be run concurrently without any interconnected dependencies, the following snippet is super useful:

{% highlight python %}
from tqdm import tqdm
from concurrent.futures import ProcessPoolExecutor, as_completed

n_jobs = 4
#Assemble the workers
with ProcessPoolExecutor(max_workers=n_jobs) as pool:
    futures = [pool.submit(work_function, x) for x in list_of_inputs]
    kwargs = {
        'total': len(futures),
        'unit': 'it',
        'unit_scale': True,
        'leave': True
    }
    #Print out the progress as tasks complete
    for f in tqdm(as_completed(futures), **kwargs):
        pass
{% endhighlight %}

The code uses `tqdm` for monitoring the process of all the jobs across all workers and `concurrent.futures` for performing concurrent work. It has the following parameters to define:

* `n_jobs`: The maximum amount of workers to have at the same time (can be smaller if your machine does not have enough resources)

* `work_function`: The function that is passed to each of the workers

* `list_of_inputs`: All inputs that has to be processed individually using `work_function`, multiple arguments can be provided, they just have to be passed as consecutive arguments to `pool.submit`

The only downside to this is that the estimates provided by `tqdm` can be a bit wonky, however the progress bar it provides is super helpful.

A sequential version of the code would be:

{% highlight python %}
from tqdm import tqdm
    
for x in tqdm(list_of_inputs):
    work_function(x)

# OR

output = [work_function(x) for x in tqdm(list_of_inputs)]  
{% endhighlight %}

This will run slower (depending on the size of the task of course), but in this case `tqdm` can provide better time estimates