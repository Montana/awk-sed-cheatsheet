## awk | sed cheatsheet


### Split compiler arg string to one options per line by pasting into terminal and hitting cntrl+d

```bash
 awk '{split($0, opts)} END {system("tput clear"); for (i in opts) print opts[i] }'
  ```

### Replace windows cr/lf with just lf. Insert a real ^M by typing ,<ctrl+v/-m>

  ```bash
 sed 's/^M//' somefile.*
  ```

### Replace path with slash separators by using a differnt separator char in sed
  
  ```bash
 sed 's;some/path/to/file;other/weird/path/2/file;' 
  ```

### Grab a tagged element from an XML file 

  ```bash
 awk '/FilePath/ {gsub("[<>]"," ", $1); split($1, tokens, " "); print tokens[2]}' montana-project.uvproj```
  ```

### Make dos paths into unix paths
  
```bash
sed 's;\\;/;g'
 ```

### Convert JSON to CSV thelong way
  
```bash
grep accel sensor-data.json | awk 'gsub(/[,:[\]]/ , " ")' | awk '{printf("%s, %s, %s, %s\n", $2, $4, $5, $6)}' > accel-data.csv
  ```

### Grab out JSON fields,  compute the delta timestamp, then plot this

  ```bash
 awk '/caledacc1/ {gsub(/[,:[\]]/ , " "); if(!lastT) lastT=strtonum($2); dt= strtonum($2) - lastT; lastT=strtonum($2); print inc++ " " dt}' ~/Downloads/emb-70.csd | gnuplot -p  -e "plot '-' with lines;"
```
  
### Convert from hex to a 16bit signed integer and plot
  
```bash
awk '{hexVal="0x" $1; numVal=strtonum(hexVal); if (numVal > 32767) numVal=numVal-65536; print i++ "   " numVal "  " hexVal; }' axu-62.dat  | gnuplot -p  -e "plot '-' with lines;"
```
  
### Poor mans histogram of the above timestamp deltas (note that there is no binning of data here)

  ```bash
 gawk '{gsub(/[,:[\]]/ , " "); if(!lastT) lastT=strtonum($2); dt= strtonum($2) - lastT; lastT=strtonum($2); print dt}' ra.csv | sort -n | uniq -c | gnuplot -p  -e "plot '-' u 2:1 with boxes;"
 ```
  
### Process data within a delimted section of a file (this process between the lines that contain `GVars1` and `--Funcs`)

  ```bash
 awk '/--- GVars/,/--- Funcs/ {if(! /---/){printf("%s:\t\t %d\n", $1, strtonum($3))} }'
  ```
### Print out an adb devices input event devices with sensor in their names

  ```bash
 adb shell cat /proc/bus/input/devices | awk '/sensor/,/event/ {d=$1; if (/N:/) {split($2,n,"=")}; if(/event/) {split($2,e,"="); print e[2] "\t" n[2];)}}'
  ```
### Print out input event devices with sensor in their names, and change their file permissions

  ```bash
 cat /proc/bus/input/devices | awk '/sensor/,/event/ {d=$1; if (/N:/) {split($2,n,"=")}; if(/event/) {split($2,e,"="); print e[2] "\t" n[2]; system("sudo chmod a+rw /dev/input/" e[2])}}'
  ```
### Update the version info in a Keil project file (be sure to git diff to make sure no extra substitutions)

  ```bash
 sed -i 's/2 1 0/2 3 0/'  someproject.uvproj 
