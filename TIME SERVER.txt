#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <error.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>



#define ERROR -1
#define MAX_CLIENTS 2
#define MAX_DATA 1024
#define BUFFER 100
#define PORT_NUMBER 60006

#define MAX(i, j) (((i) > (j)) ? (i) : (j))




 char wrong_req[1000]="INCORRECT REQUEST\n";  
 char close_server[1000]="\n-----------------GOOD BYE-----------------\n";  
 int finish=1;

int main(int argc, char const *argv[])
{


     //-----------
     FILE *output;
     FILE *output2;
     FILE *output3;
     FILE *output4;
     FILE *output5;

     char buffer[BUFFER]; // Setting up buffers for use of linux bash commands
     char buffer2[BUFFER];
     char buffer3[BUFFER];
     char buffer4[BUFFER];
     char buffer5[BUFFER];


     int chars_read;


     memset(buffer,'\0',sizeof(buffer)); // Clearing buffers from trash data
     memset(buffer2,'\0',sizeof(buffer2));
     memset(buffer3,'\0',sizeof(buffer3));
     memset(buffer4,'\0',sizeof(buffer4));
     memset(buffer5,'\0',sizeof(buffer5));


     


 
    //-----------
    char available_requests [1000]="Avilable Requests on Server:\n[GET_TIME]\n[GET_DATE]\n[GET_TIME_DATE]\n[GET_TIME_ZONE]\n[GET_DAY_OF_WEEK]\n[CLOSE_SERVER]\n\nPlease enter your date request : \n";




    int value;
    struct sockaddr_in server;
    struct sockaddr_in client;
    int sock;
    int new;
    int sockaddr_len= sizeof(struct sockaddr_in);
    int data_len;
    char data[MAX_DATA];



    if ((sock=socket(AF_INET,SOCK_STREAM,0))==ERROR)
    {
        perror("server socket: ");
        exit(-1);
    }

    server.sin_family= AF_INET;
    server.sin_port =htons(PORT_NUMBER); // Open up a connection in the port 60006
    server.sin_addr.s_addr= INADDR_ANY;
    bzero(& server.sin_zero,8);


    if ((bind(sock,(struct sockaddr *)&server,sockaddr_len))==ERROR)
    {
        perror("bind : ");
        exit(-1);
    }

    if ((listen(sock,MAX_CLIENTS))==ERROR)
    {
        perror("listen");
        exit(-1);

    }

    while (finish) // while loop to wait for connections
    {
       if ((new =accept(sock,(struct sockaddr *)&client,&sockaddr_len))==ERROR)
       {
        perror("accept");
        exit(-1);
        }

        printf("A Client Connected From Port %d and IP %s\n",ntohs(client.sin_port),inet_ntoa(client.sin_addr));

        data_len=1;
        char welcome_mesg[1000]="\n\n-----------------Welcome to Date Server-----------------\n";
        send(new,welcome_mesg,1000,0);
        send(new,available_requests,1000,0);
        
        while (data_len) // while loop that is going to be used to listen to commands
        {
            
            data_len= recv(new ,data,MAX_DATA,0);
            data[data_len - 1] = 0;
            data[data_len - 2] = 0; // to get rid of string end characters
            if (data_len)
            {  

                  if(memcmp( data, "GET_DATE", MAX(strlen(data),strlen("GET_DATE")))== 0){ // GET_DATE command

                                char cmd1[1024];
                                snprintf(cmd1, sizeof(cmd1), "date +%c%c.%cm.%cY", '%','e','%','%');

                                output=popen(cmd1,"r");
                                if (output!=NULL)
                                {
                                    int count =1;
                                    while (fgets(buffer,BUFFER-1,output)!=NULL)
                                    {
                                    send(new,buffer,BUFFER,0);
                                    count++;
                                    }
                                    pclose(output);
                                }

                 }
                 else if( memcmp( data, "GET_TIME", MAX(strlen(data),strlen("GET_TIME")))== 0)  { // GET_TIME command
                
                    char cmd4[1024];
                    snprintf(cmd4, sizeof(cmd4), "date +%cT", '%');

                    output4=popen(cmd4,"r");
                    if (output4!=NULL)
                    {
                        int count =1;
                        while (fgets(buffer4,BUFFER-1,output4)!=NULL)
                        {
                        send(new,buffer4,BUFFER,0);
                        count++;
                        }
                        pclose(output4);
                    }
                 } 

                  else if(memcmp( data, "GET_TIME_DATE", MAX(strlen(data),strlen("GET_TIME_DATE" )))== 0){ // GET_TIME_DATE

                            char cmd2[1024];
                            snprintf(cmd2, sizeof(cmd2), "date +%cH:%cM:%c%c,%c%c.%cm.%cY", '%','%','%','S','%','e','%','%');

                            output2=popen(cmd2,"r");
                            if (output2!=NULL)
                            {
                                int count =1;
                                while (fgets(buffer2,BUFFER-1,output2)!=NULL)
                                {
                                send(new,buffer2,BUFFER,0);
                                count++;
                                }
                                pclose(output2);
                            }
                
               
                
                }
                else if(memcmp( data, "GET_TIME_ZONE", MAX(strlen(data),strlen("GET_TIME_ZONE" )))== 0){ // GET_TIME_ZONE

                                char cmd3[100];
                                snprintf(cmd3, sizeof(cmd3), "date +%c:z", '%');

                                output3=popen(cmd3,"r");
                                if (output3!=NULL)
                                {
                                    int count =1;
                                    while (fgets(buffer3,BUFFER-1,output3)!=NULL)
                                    {
                                    send(new,buffer3,BUFFER,0);
                                    count++;
                                    }
                                    pclose(output3);
                                }
                
                
                }
                
                
               
                else if(memcmp( data, "GET_DAY_OF_WEEK", MAX(strlen(data),strlen("GET_DAY_OF_WEEK")))== 0){ // GET_DAY_OF_WEEK

                  char cmd5[100];
                                snprintf(cmd5, sizeof(cmd5), "date +%cA", '%');
                                output5=popen(cmd5,"r");
                                if (output5!=NULL)
                                {
                                    int count =1;
                                    while (fgets(buffer5,BUFFER-1,output5)!=NULL)
                                    {
                                    send(new,buffer5,BUFFER,0);
                                    count++;
                                    }
                                    pclose(output5);
                                }
                
                
                
                }
                else if(memcmp( data, "CLOSE_SERVER", MAX(strlen(data),strlen("CLOSE_SERVER" )))== 0){ // CLOSE_SERVER

                 send(new,close_server,1000,0);
                
                printf("CLient disconnected \n");
                close(new);
                close(sock);
                finish=0;
                break;
                
                }
                
                else{
                send(new,wrong_req,1000,0);
            }
        }
        
       
    }
     
    
    return 0;
    }
}
