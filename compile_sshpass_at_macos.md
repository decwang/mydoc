wget http://sourceforge.net/projects/sshpass/files/sshpass/1.05/sshpass-1.05.tar.gz  
tar xvzf sshpass-1.05.tar.gz  
cd sshpass-1.05
./configure --prefix=/usr/local/Cellar/sshpass/1.05  
make  
sudo make install  