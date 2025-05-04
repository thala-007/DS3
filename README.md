
open terminal in home:
Extract jar file

wget https://sourceforge.net/projects/mpjexpress/files/latest/download -O mpj.zip
unzip mpj.zip -d mpj


Your project should look like this:

DistributedSumProject/

├── mpj/                     ← Contains extracted mpj-v0_44

│   └── mpj-v0_44/

├── src/

│   └── DistributedSum.java

└── run.sh                   ← (You’ll create this script to run your program)



---

✅ Step 1: Set MPJ_HOME and update PATH (if not already done)

1. Open .bashrc:

nano ~/.bashrc


2. Add these lines at the end:

export MPJ_HOME=~/DistributedSumProject/mpj/mpj-v0_44
export PATH=$MPJ_HOME/bin:$PATH


3. Save and exit (Ctrl + O, then Enter, then Ctrl + X).


4. Reload bashrc:

source ~/.bashrc


5. Verify:

echo $MPJ_HOME

It should show:

/home/pvg/DistributedSumProject/mpj/mpj-v0_44




---

✅ Step 2: Write your Java code

Make sure your file DistributedSum.java in src/ folder looks like this:

import mpi.*;

public class DistributedSum {
    public static void main(String[] args) throws MPIException {
        MPI.Init(args);
        int rank = MPI.COMM_WORLD.Rank();
        int size = MPI.COMM_WORLD.Size();

        int[] data = new int[100];
        for (int i = 0; i < 100; i++) {
            data[i] = i + 1;
        }

        int chunkSize = data.length / size;
        int start = rank * chunkSize;
        int end = (rank == size - 1) ? data.length : start + chunkSize;

        int local_sum = 0;
        for (int i = start; i < end; i++) {
            local_sum += data[i];
        }

        int[] global_sums = new int[size];
        MPI.COMM_WORLD.Allgather(new int[]{local_sum}, 0, 1, MPI.INT, global_sums, 0, 1, MPI.INT);

        if (rank == 0) {
            int total = 0;
            for (int sum : global_sums) {
                total += sum;
            }
            System.out.println("Total Sum: " + total);
        }

        MPI.Finalize();
    }
}


---

✅ Step 3: Compile the Java file

From inside src/:

javac -cp $MPJ_HOME/lib/mpj.jar DistributedSum.java

If it gives no error, you’re good.


---

✅ Step 4: Run using mpjrun.sh

Back in the root directory (DistributedSumProject), create a script run.sh:

nano run.sh

Paste this in:

#!/bin/bash
$MPJ_HOME/bin/mpjrun.sh -np 4 -cp src DistributedSum

Make it executable:

chmod +x run.sh

Run your program:

./run.sh

You should see the output:
Total Sum: 55
