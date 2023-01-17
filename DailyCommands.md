#Delete History<br>
history -d 15154

#delete histiry in range<br>
for h in $(seq 15154 15186); do history -d 15154; done

#read and write access to file<br>
chmod 777 fileName

#To trnasfer file from local to server via ssh<br>
scp /<..path..>/<..jarname....>SNAPSHOT.jar <..userName..>@<..serverHostName..>:/home/<..userName..>/<..fileName..>SNAPSHOT.jar
