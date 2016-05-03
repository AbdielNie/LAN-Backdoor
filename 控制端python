from Tkinter import *
import socket
import threading
import random

ENDCODE = random.randrange(100,999)

#本机监听地址
recevAddr = (socket.gethostbyname(socket.gethostname()), 8889)
#初始化监听组件
listener = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
listener.bind(recevAddr)
class newOrder:
	cout = 0
	viewDir = {}
	ss = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
	target = ('192.168.134.1',8888)
	def getViewDir(self):
		return newOrder.viewDir
	def getSock(self):
		return newOrder.ss
	#创建新弹窗
	def createView(self):
		newOrder.cout += 1
		self.cout = newOrder.cout
		#弹窗
		topl = Toplevel()
		topl.resizable(width = False, height = False)
		#结果显示
		Label(topl, text = '回显结果').pack()
		resultFrame = Frame(topl)
		resultFrame.pack()
		#滑块
		textScroll = Scrollbar(resultFrame)
		textScroll.pack(side = RIGHT, fill = Y)
		result = Text(resultFrame)
		result.config(height = 20)
		newOrder.viewDir[str(newOrder.cout)] = result
		result.pack(side = LEFT, fill = Y)
		result.config(yscrollcommand = textScroll.set)
		textScroll.config(command = result.yview)
		#命令输入
		Label(topl, text = '输入命令').pack()
		order = Text(topl)
		order.config(height = 10)
		order.pack()
		self.order = order
		#执行按钮和模式选项
		BottomFrame = Frame(topl)
		BottomFrame.pack(fill = X)
		checkvar = IntVar()
		sw = Checkbutton(BottomFrame,text = '代码模式(忽略换行)', variable = checkvar, onvalue = 1, offvalue = 0)
		sw.pack(side = LEFT)
		self.mod = checkvar
		act = Button(BottomFrame, text = 'start', command = self.sendMsg, width = 10, bg = 'grey')
		act.pack(side = RIGHT)
		act.bind('<Destroy>',self.clean)
		topl.mainloop()
	#清理弹窗词典
	def clean(self, event):
		del newOrder.viewDir[str(self.cout)]
	#发送标记有窗口信息的完整信息
	def sendMsg(self):
		tmpMsg = self.order.get(0.0,END)
		if self.mod.get():
			tmpMsg = tmpMsg.replace('\n','')
		Msg = str(self.cout) + '->' +tmpMsg
		newOrder.ss.sendto(Msg, newOrder.target)
		print(tmpMsg)
		
	
def createOrder():
        view = newOrder()
        view.createView()


def msgGot():
	while True:
		data, addr = listener.recvfrom(1024*16)
		try:
			viewNum = data.split('->')[0]
			tmpNum = int(viewNum)
			if tmpNum == ENDCODE:
				listener.close()
				return
			elif tmpNum not in range(1,99):
				continue
		except:
			continue
		tmprevMsg = data[len(viewNum)+2:]
		tmpInstence = newOrder()
		viewDir = tmpInstence.getViewDir()
		try:
			textContainer = viewDir[viewNum]
		except:
			continue
		textContainer.insert(END,tmprevMsg.decode('GBK'))

		

root = Tk()
root.title("添加新指令")
root.resizable(width = False, height = False)
root.geometry('200x100+300+200')
comfirm = Button(root, text = 'Add', command = createOrder)
comfirm.pack(side = BOTTOM)
#监听来信
threadIn = threading.Thread(target = msgGot, args = ())
threadIn.start()
root.mainloop()
#发送给本机一个结束信号
lastOperate = newOrder().getSock()
lastOperate.sendto(str(ENDCODE)+'->',recevAddr)
lastOperate.close()


