This will be a set of examples that utilize AWK functions.

The best part about BASH is that it treats all text as numbers and vice versa.

That would normally sound terrible, and is in many programming settings, the quick-and-dirty
nature of the CLI makes this exactly what I like about it. 

So much of mod/sim work is about manipulating text, so most scripts/codes for pre-/post-analysis are boilerplate IO. 

AWK is excellent at this text manipulation, and also has great uses just as a command line tool.

Lets go through the basics:

Like head, tail, cat, etc., data in awk is manipulated as effectively rows and columns of values

>awk '{print($1,$2,$3)}' File

Nice, just gives us the columns 1,2,3 out of our file. Combos well when you are listing folders;

>ls -ltrh | awk '{print($10)}'

Natively, the field seperater is a space, but you can define your own with a flag

> awk -F "," print($1,$2,$3) File

Other useful things to know;

> awk '{print($0)}'

Prints all columns

> awk '{print(NR)}'

Prints line numbers

> awk '{print(NF)}'

Prints number of columns on each line

For example;

>alias width='comp_width'
>function comp_width () {
>        input=$1
>        awk '{print(NF)}' $1 | sort -nk1,1  | tail -n1
>}
 
Gives you the max width in a file, and is super fast since its native in bash

Now a I said before AWK processes a file line by line, so you can set up conditional prints based on data in a line. 

Everything in the {} is a condition to print, left blank it is True.

>awk '{if(NF==3)print($0)}'

> ls -ltrh | awk -F ".xyz" '{print($0)}' | awk -F "File" '{print($1)}' | awk '{if($1<10000)print($1)}'


#Ok now for some fun stuff, you can create arrays to manipulate text before printing;
## flip File

>alias flip='comp_transpose'
>function comp_transpose () {
>        input=$1
>        awk '{for (i=1; i<=NF; i++) a[i]=a[i](NR!=1?FS:"")$i} END {for (i=1; i in a; i++) print(a[i])}' $input
>}

Reads as: for each row iterate up to the number of columns, at the END print out each row as a column.

## head -n1 File | flip | awk '{print(NR,$0)}'

Quickly figure out what column each data is in

Heres a nice use case, find all matching files and use them in a loop.

>files=`ls -ltrh *.xyz | awk '{print($10)}' | flip`
>for i in $files; do lines=`wc -l $i | awk '{print($1)}'` ; echo "File" $i "has " $lines " lines." ; done

#More Examples
Ok how about a cool function to drop into your own bash profile that uses AWK
>alias who='comp_who'
>function comp_who () {
>        echo "USER        NUM_JOBS  NUM_NODES  RUNNING_NOW" 
>        userdata=`squeue | awk '{print($4)}' | sort -u | awk '{for (i=1; i<=NF; i++) a[i]=a[i](NR!=1?FS:"")$i} END {for (i=1; i in a; i++) print(a[i])}'`
>        echo > tmp
>        for peruser in $userdata
>        do
>         njobs=`squeue   | grep $peruser | wc -l`
>         nnodes=`squeue  | grep $peruser | awk '{sum +=$7}END{print sum}'`
>         rnnodes=`squeue | grep $peruser |awk '{if($5=="R") print($0)}' | awk '{sum +=$7}END{print sum}'`
>         printf "%s\t\t%d\t%d\t%d\n" $peruser $njobs $nnodes $rnnodes
>         echo $njobs $nnodes $rnnodes >> tmp
>        done
>        echo "--------------------------------------"
>        totnjobs=`awk '{sum +=$1}END{print sum}' tmp`
>        totnnodes=`awk '{sum +=$2}END{print sum}' tmp`
>        totrnodes=`awk '{sum +=$3}END{print sum}' tmp`
>        printf "%s\t\t%d\t%d\t%d\n" "TOTAL" $totnjobs $totnnodes $totrnodes
>        rm -f tmp
>}

#Compute the average value down a column of data
## avg File 2
>alias avg='comp_avg'
>function comp_avg () {
>        input=$1
>        col=$2
>        awk '{ if(NF>0) (sum+=$'"$col"' n++)} END { print(n,sum,sum/n); }' $input
>}

#Compute the standard deviation down a column of data
## std File 2
>alias std='comp_std'
>function comp_std () {
>        input=$1
>        col=$2
>        avg=`awk '{ if(NF>0) (sum+=($'"$col"'/10000.) n++)} END { print(sum/n); }' $input`
>        std=`awk '{ if(NF>0) (sum+=(($'"$col"'/10000.-'"$avg"')**2.0) n++)} END { print((sum/n)); }' $input`
>        finalavg=`awk '{print('"$avg"'*10000)}' $input | tail -n1`
>        finalstd=`awk '{print(('"$std"'*10000)**0.50)}' $input | tail -n1`
>        echo "AVG: " $finalavg "STD: " $finalstd
>}

#Generate random integers between two values you define
## randint 1 10
>alias randint='comp_randint`
>function comp_rnadint () {
>        min=$1
>        max=$2
>       `awk -v min=${min} -v max=${max} 'BEGIN{srand(); print int(min+rand()*(max-min+1))}'`
>}

#If you want to bring in variables from the CLI, need to break the single quotes.
>break1=`head -n1 Counts | tail -n1 | awk '{print($1)}'`
>break2=`head -n2 Counts | tail -n1 | awk '{print($1-'"${break1}"')}'`
