#Delete History
history -d 15154

#delete histiry in range
for h in $(seq 15154 15186); do history -d 15154; done
