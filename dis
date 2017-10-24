//============================================================================
// Name        : testsocketzhuang.cpp
// Author      : han
// Version     :
// Copyright   : Your copyright notice
// Description : Hello World in C++, Ansi-style
//============================================================================
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <sys/time.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <iostream>
#include <map>
#include <fstream>
#include <stdexcept>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
using namespace std;
map<int,int> number_pid;

int main(){

	ofstream outfile;
    outfile.open("mydeep.txt");


	int server_sockfd = -1, client_sockfd = -1;     //服务监听socket
	unsigned int server_len, client_len;
	struct sockaddr_in server_address;
	struct sockaddr_in client_address;
	int fpid = -1;                       //fork进程id
	int result = -1;                          //select结果
	fd_set readfds, testfds;//select的socket集合

	server_sockfd = socket(AF_INET, SOCK_STREAM, 0);//socket产生套接字

	server_address.sin_family = AF_INET;
	server_address.sin_addr.s_addr = htonl(INADDR_ANY);
	server_address.sin_port = htons(5000);
	server_len = sizeof(server_address);

	int bind_return = bind(server_sockfd, (struct sockaddr *)&server_address, server_len);//绑定服务套接字

    while(bind_return == -1)
    {
    	printf("bind error,Please cout new port number\n");
    	int bind_port = 5000;
    	scanf("%d",&bind_port);
    	server_address.sin_port = htons(bind_port);
    	bind_return = bind(server_sockfd,(struct sockaddr *)&server_address,server_len);
    }

	listen(server_sockfd, 10);//监听套接字，三次握手队列5


    FD_ZERO(&testfds);
	FD_ZERO(&readfds);
	FD_SET(server_sockfd, &readfds);

	while(1){
		char buffer[1024];
		int fd;
		int nread;

		testfds = readfds;

		printf("server waiting\n");
        outfile <<"server waiting\n";
		result = select(FD_SETSIZE, &testfds, (fd_set *)0, (fd_set *)0, (struct timeval *)0);//select监听，阻塞状态，超时为无限
        //block

        printf("%d\n",result);
		if(result < 1){   //select错误
			perror("server5");
			for(fd = 0; fd< FD_SETSIZE; fd++)
			{
				if(FD_ISSET(fd, &testfds))
				{
				outfile<<fd<<endl;
				}
			}

			// send error to yunwei
			continue;
		}


		for(fd = 0; fd< FD_SETSIZE; fd++){//轮询套接字，发现处于读状态的socket
			if(FD_ISSET(fd, &testfds)){//是否在读状态socket
				if(fd == server_sockfd){
					client_len = sizeof(client_address);
					client_sockfd = accept(server_sockfd, (struct sockaddr *)&client_address, &client_len); //三次握手提取socket
					FD_SET(client_sockfd, &readfds);
					printf("addig client on fd %d\n", client_sockfd);
				}
				else {
					ioctl(fd, FIONREAD, &nread);
					if(nread == 0){
						close(fd);
						FD_CLR(fd, &readfds);
						printf("removing client on fd %d\n", fd);
						outfile<< "removing client on fd" <<fd<<"\n";
					}
					else
					if(nread > 0)
					{

						read(fd, buffer, nread);//读取信息
						printf("serving client on fd %d\n", fd);
						outfile << "serving client on fd" << fd << "\n";
						char* lintem[9]; //命令字符串
						char tempstring[10] = "push";
						lintem[0] = tempstring;
						int ber = 1;
						char* key_p = NULL;
						char* buffer_p = buffer;
                        while((key_p=strsep(&buffer_p," "))!=NULL)
                        {
                        	if(ber==2 && key_p[0]=='1')
                        	{
                        		break;
                        	}

                             if(ber == 6){
                            	 char wws[]="554";
                            	 lintem[ber] = wws;
                           } else {
                        	   lintem[ber] = key_p;
                           }
                             printf("%s\n",lintem[ber]);
                             outfile << lintem[ber] << "\n";
                             ++ber;
                        }

                        if(ber == 2)//如果只有两个参数，可能是失败，或者停止推流。
                        {
                        	printf("############%s\n",lintem[1]);
                        	map<int,int>::iterator iter = number_pid.find(atoi(lintem[1]));
                        	if(iter!=number_pid.end())
                        	{
                        		int status;

                        		if(waitpid(iter->second,&status,WNOHANG) == 0)//存在该进程
                        		{
                        			printf("pid : %d\n",iter->second);
                        			int retval = kill(iter->second,SIGKILL);//关闭进程
                        			if(retval == -1)
                        			{   outfile<<"error:" <<iter->second <<"\n";
                        				printf("%d error",iter->second);
                        				waitpid(iter->second,&status,0);
                        				char a[] = "7";
                        				write(fd,a,sizeof(a));
                        				close(fd);
                        				FD_CLR(fd, &readfds);
                        			}
                        			else
                        			{
                        				waitpid(iter->second,&status,0);
                        				printf("%d kill\n",iter->second);
                        				char a[] = "6";      //kill success
                        				write(fd,a,sizeof(a));
                        				close(fd);
                        				FD_CLR(fd, &readfds);
                        			}
                        			number_pid.erase(iter);
                        		} else

                        		{
                        			printf("watipid bu cunzai :%d\n",iter->second);
                        			char a[] = "8";
                        			write(fd,a,sizeof(a));
                        			close(fd);
                        			FD_CLR(fd, &readfds);
                        			number_pid.erase(iter);
                        		}

                        	} else
                        	{
                        		printf("stream name bu chuz:%d\n",iter->first);
                        		char a[] = "8";
                        		write(fd,a,sizeof(a));
                        		close(fd);
                        		FD_CLR(fd, &readfds);
                        	}
                        	char patharray[50] = "/usr/local/movies/";
                        	    char *path = patharray;
                        	    char streamName[20];
                        	    strcpy(streamName,lintem[1]);
                        	    strcat(streamName,".sdp");
                        	    strcat(path,streamName);
                        		if(-1 != access(path,0))
                        	    {
                        			if(-1 !=remove(path))
                        			{
                        				printf("delete %s\n",streamName);
                        			}
                        	    }


                        }
                        else
                        {map<int,int>::iterator iter = number_pid.find(atoi(lintem[1]));
                    	if(iter==number_pid.end())
                    	{

                            char stee[10];
                            sprintf(stee,"%d",fd);
    						lintem[7] = stee;
    						lintem[8] = NULL;
    						fpid = fork();
    						if(fpid==0)
    						{
    							execv("./push",lintem);
    							printf("command ls is not found, error code: %d(%s)\n",errno, strerror(errno));
                                outfile << errno;
    							char a[] = "4";
    							write(fd,a,sizeof(a));
    							close(fd);
    							FD_CLR(fd, &readfds);
    							exit(0);
    						}
    						else
    							if(fpid==-1)
    						{
        							char a[] = "5";
        							write(fd,a,sizeof(a));
        							close(fd);
        							FD_CLR(fd, &readfds);
    						}
    						else
    						{

                                for(int i = 0;i<10;i++)
                                {
                                	if(lintem[1][i]=='.')
                                	{
                                		lintem[1][i] = '\0';
                                	}
                                }
                                printf("%s\n",lintem[1]);
    							number_pid.insert(pair<int,int>(atoi(lintem[1]),fpid));
                                printf("sucess\n");
                                signal(SIGCLD, SIG_IGN);
    						}

                        }
                    	else
                    	{
                    		char a[] = "0";
                    		write(fd,a,sizeof(a));
                    		close(fd);
                    		FD_CLR(fd, &readfds);
                    	}





					}
					}
				}
			}
		}

	}


}



























