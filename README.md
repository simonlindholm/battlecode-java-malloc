# battlecode-java-malloc

A hack for `LD_PRELOAD`-replacing malloc for JNI.
Effectiveness is untested, and it does double Rust memory usage
(because of moving GC, JNI cannot get direct pointers to Java-managed memory).

To build your own `preload.so` (not necessary):

- Run `sudo ./run.sh` to get the battlecode docker container running
- Open a new terminal, and create a shell within docker by doing: `sudo docker exec -it $(sudo docker ps -q) /bin/sh`
- Create a shell within the nested battlebaby container: `docker run -v /:/oldroot -it battlebaby /bin/sh`
- Copy `replace.cpp` into the container by copying it from `/oldroot/player/` (the directory from where you ran ./run.sh)
- Run the following from within the nested container:
```bash
g++ -O2 -Wall -Wextra -shared -fPIC -o replace.so replace.cpp -I /opt/jdk/include/ -I /opt/jdk/include/linux/ -std=c++11 -L /opt/jdk/lib/server/ -ljvm -Wl,-rpath,/opt/jdk/lib/server/
```
- Copy the resulting `replace.so` out to `/oldroot/player/`

To use it, add `LD_PRELOAD=./preload.so` to the `java` command in your bot's `run.sh`. E.g.:
```bash
LD_PRELOAD=./replace.so java -Xms256m -Xmx256m -classpath .:../battlecode/java Player
```
The -Xms and -Xmx flags tell Java never to allocate more than 256 MB memory, and are very likely to be helpful.
