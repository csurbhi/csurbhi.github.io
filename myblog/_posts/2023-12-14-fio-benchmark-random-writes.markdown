# FIO benchmarks for random writes on Linux:
  

FIO is an open source storage benchmark that runs on many Operating systems. It is used to measure a file/device’s read/write bandwidth and IOPS. In this article we look at the FIO options on Linux.

  

FIO runs with command line options or a configuration file with the options specified. It performs synchronous or asynchronous I/O to some file/device using the specified options. The options decide if the I/O is buffered or is directed to the underlying device.

Synchronous and asynchronous reads have to go to the device in case the requested read data is not buffered. *However, writes can complete without going to the device; the data is written in the page cache and the userspace write can complete.* *In this case, we see the bandwidth and iops reflect the i/o to the DRAM rather than the backend device.*

  
FIO has a lot of parameters, we can look at some of the relevant and interesting parameters:

1.  **ioengine**: this specifies the mode of the I/O. I/O can be submitted synchronously or asynchronously. The ioengine for synchronous I/O is “sync” and for asynchronous I/O there are quite a few options: “libaio”, “io_uring”, ”posix_aio” etc. Note that on Linux, libaio is older than io_uring.  

  

2.  **direct**: This specifies the kind of the i/o in terms of location. If direct = 0, the i/o is buffered and comes out of the buffer/page cache. If direct = 1, this is equivalent to the O_DIRECT flag. Note that when ioengine is “libaio”, direct has to be 1 or buffered has to be 0. Libaio does not support buffered i/o on Linux. 

 

3.  **buffered**: This is opposite to the direct flag. By default this is 1 or “true”.
    

 
4.  **iodepth**: This specifies the number of asynchronous I/O that are kept in flight against a particular file/device. This parameter matters only when the specified ioengine is NOT synchronous.
    
iodepth does not specify how many ios can be submitted together at a time. This is specified by the iodepth_batch option we see next.

  

5.  **iodepth_batch** - This suggests how many asynchronous I/Os can be submitted together.
    

 The difference between iodepth and iodepth_batch can be expressed as as follows:

	resume_queueing()
	{
		while (queued_requests < iodepth) {
			queue_request(io_request);
			queued_requests ++;
			if (queued_requests >= iodepth_batch) {
				for(i=0; i<iodepth_batch; i++) {
					submit_asynchronously(io_request);
					queued_requests --;
				}
			}
		} /* end of while loop */
		wait_for_completion_of_atleast_one_request();
		goto resume_queueing;
	}


***Note that iodepth is not the same as the queue depth of a storage device.*** It is possible to ensure that the backend storage device receives maximum requests it can hold in it’s hardware queue. This can be done for example by making sure that the kernel’s io scheduler always has enough I/Os to send to the backend device. To do so, we can specify iodepth to be a number much larger that the actual queue depth of the backend storage device. We can next specify the iodepth_batch to match that of the known “queue depth” on the storage device hardware. This way, FIO can generate a larger number of I/Os , they can be in-flight (or incomplete). This ensures that fio will always send to the kernel (thus the ioscheduler) a constant stream of I/Os. In this way, the backend device will always have maximum number of I/Os queued in the hardware queues.

However, FIO cannot ensure  that the storage device's queue will always hold the specified "iodepth" I/Os. 
The combination of "iodepth" and "iodepth_batch" can ensure that the maximum i/o’s (but not the minimum) submitted to the device through this particular instance of FIO will always be less than or equal to the “iodepth”.  For example: Say iodepth is 4. FIO issues at max 4 ios and waits for 1 to complete. The instance 1 i/o completes FIO starts generating and queueing i/o again. Generating, queueing and process scheduling (FIO can get scheduled out and in) introduces a delay during which the storage backend can complete all the queued I/O. At this point the number of requests in the I/O queue can become 0.


**FIO Reads and Writes:**

In order to examine the performance of the backend device, the “direct” flag should always be 1 (and/or the “buffered” flag should be 0). Not doing so, may provide the performance of reads/writes to the page cache. Unless you want to demonstrate the efficiency of your cache, FIO should use “direct=1” to showcase the true device performance.

-------------------------------

These are the two ways **synchronous I/O** can be configured in FIO to ensure that the i/o is to/from the backend device.

	ioengine = sync,
	direct = 1
	buffered =0

OR

	ioengine = libaio/io_uring
	buffered = 0
	direct = 1
	iodepth = 1

-------------------------------

Similarly, **asynchronous I/O** can be configured as follows:

	ioengine = libaio
	direct = 1
	buffered = 0
	iodepth = "x"
	io_batch = "y" [where y < x ]

OR

	ioengine = io_uring
	direct = 1
	buffered = 0
	iodepth = "x"
	iodepth_batch = "y" [where y < x ]

-------------------------------

To benchmark the random write performance we need to set two more parameters at least. They are as follows:
1.  **readwrite**=randwrite
	  This indicates that FIO will issue random writes. The other option could be randrw or randread that says that FIO will issue random reads and writes or random reads respectively.

2.  **random_distribution**: FIO generates random I/O with a distribution specified by this parameter. The values for this can be “zipf”, “zoned”, “pareto”, and “normal”. These can parameters can be augmented with a second optional float value. It allows one to set base of distribution in non-default place, giving more control over most probable outcome. This value is in range [0-1] which maps linearly to range of possible random values. Defaults are: 0.3 for pareto, 1.2 for zipf, and 0.5 for normal. As an example: If you wanted to use zipf with a theta of 1.2 centered on 1/4 of allowed value range, you would use random_distribution=zipf:1.2:0.25.

This blog will not talk about the difference between the random distributions, google can provide plenty of resources for that. Here, we only look at ways to configure FIO to achieve the distribution of our choice. 

 FIO comes with a **fio-genzip** tool that shows the random distribution associated with the optional theta// for zipf, pareto and normal random distribution.

	   $ fio-genzipf -h shows the following:
    	genzipf: test zipf/pareto values for fio input
    	-h This help screen
    	-p Generate size of data set that are hit by this percentage
    	-t Distribution type (zipf, pareto, or normal)
    	-i Distribution algorithm input (zipf theta, pareto power,
    		or normal % deviation)
    	-b Block size of a given range (in bytes)
    	-g Size of data set (in gigabytes)
    	-o Number of output rows
    	-c Output ranges in CSV format

    The default theta value for zipf is 1.2. Running, fio-genzipf for “zipf” with the default theta gives you the following output (partial output pasted here).
    $ fio-genzipf -t zipf
    Generating Zipf distribution with 1.200000 input and 500 GiB size and 4096 block_size.
    Rows 		Hits % 		Sum % 		# Hits 			Size    
    -----------------------------------------------------------------------    
    Top 	5.00% 	94.83% 	94.83% 	124301859 474.17G
    |-> 	10.00% 	1.02% 	95.86% 	1342527 	5.12G
    |-> 	15.00% 	0.59% 	96.45% 	769476 		2.94G    
    |-> 	20.00% 	0.39% 	96.84% 	514618 		1.96G    
    |-> 	25.00% 	0.36% 	97.20% 	468626 		1.79G    
    |-> 	30.00% 	0.30% 	97.50% 	394525 		1.50G
    -----------------------------------------------------------------------    

This is what fio-genzipf code does:

	main()
	{
		total_blks = disk_size_in_bytes / blk_size_in_bytes;
		for(i=0; i<total_blk; i++) {
			nr = generate_next_random_nr( zipf ); /* zipf can be replaced by pareto or gauss */
			found = check_in_table(nr);
			if (found) {
				found->hits++;
			} else {
				insert_in_table(nr);
			}
		}
		descending_sort_table_on_hits();
		/* by default output rows is 20, this is configurable */
		divide_table_in_rows(20, total_blks); 
	}

	divide_table_in_rows(output_rows, table, total_blks) 
	{
		Interval = table.rows/output_rows;
		for(i = 0; i<table.rows; i++) {
			hits = hits + table[i].hits;
			total_hits = total_hits + hits;
			If (i == next_interval) {
				next_interval = next_interval + interval;
				hits = 0;
				Output.hits = (hits/total_blks) * 100;
			}
		}
	}

 
To summarize, the *fio-genzipf code basically sorts based on hits and presents a summary* that by default is for every 5% hit intervals. The 2nd column shows the percentage hits received by an interval. For example, the above output says that 5% of the random LBAs receive 95% hits.

  
For zipf, *a higher theta gives smaller tail values*. **To achieve a 80/20 distribution, specify theta to be 0.9517**
  

	$ fio-genzipf -t zipf -i 0.9517
	Generating Zipf distribution with 0.951700 input and 500 GiB size and 4096 block_size.
	Rows Hits % Sum % # Hits Size
	-----------------------------------------------------------------------
	Top 5.00% 68.20% 68.20% 89396997 341.02G
	|-> 10.00% 5.40% 73.60% 7078075 27.00G
	|-> 15.00% 3.61% 77.22% 4733348 18.06G
	|-> 20.00% 2.78% 80.00% 3647538 13.91G



**To achieve a 90/10 distribution, specify a theta to be 1.0962**
	$ fio-genzipf -t zipf -i 1.0962	
	Generating Zipf distribution with 1.096200 input and 500 GiB size and 4096 block_size.
	
	Rows Hits % Sum % # Hits Size
	-----------------------------------------------------------------------
	Top 5.00% 87.80% 87.80% 115084376 439.01G
	|-> 10.00% 2.20% 90.00% 2885564 11.01G


