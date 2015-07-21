#!/usr/bin/python

# Title: QProcess Control via GUI
# Author: Aleksandar Vladimirov Atanasov
# Description: Launches and terminates a detached process using a toggable push button

# Notes:
#   - "rosnode kill -a" kills all nodes

# QProcessControl command format (type: dictionary):
# - 'command' - command to be executed (roslaunch | rosrun)
# - 'pkg' - ros package (curently no check is made if the package exists or not)
# - 'node' - ros node or one|multiple launch files (curently no check is made if the node or launch files exists)
#	     in case of rosrun command 'node' takes a single string as value
#	     in case of roslaunch command 'node' takes a list of strings as values (one or multiple launch files)
# - 'pid' - file where the PID of the executed command is stored; this file is used to restore the state of the GUI even after
#	    the application has quit or has crashed

import sys, subprocess, os
from signal import SIGINT
from PyQt4.QtGui import QApplication, QWidget, QPushButton, QHBoxLayout
from PyQt4.QtCore import QProcess

def check_pid(pid):
	try:
		os.kill(pid, 0)
		return True
	except OSError:
		return False

class QProcessControl(QWidget):

    def __new__(cls, command):
	# Check if the command dictionary has the required size
	if len(command) != 4:
		print('Error: Incorrect argument passed to constructor. Expecting dictionary format:\n' +
		'{\n\t"command"\t: "<roslaunch|rosrun>",' +
		'\n\t"pkg"\t\t: "<pkg_name>",' +
		'\n\t"node"\t\t: "<nodeFile|[launchFile(s)]>"' +
		'\n\t"pid"\t\t: "<pid_fila_path>' +
		'\n}')
		return None
	print 'List found'
	# Check if all required keys are present in the command dictionary
	for key in ['command', 'pkg', 'node', 'pid']:
		if key not in command:
			return None
	print 'Integrity check for list passed'
	# Check if the "command" is a known one
	if command['command'] not in ['roslaunch', 'rosrun']:
		print 'Error: Unknown command. Choose between "roslaunch" and "rosrun"'
		return None
	print 'Correct command'
	# If "roslaunch" was selected as "command", check if "node" contains a list of one or more values (the roslaunch command can start multiple launch files at once)
	if command['command'] == 'roslaunch' and not isinstance(command['node'], list):
		print 'Error: Expecting list of one or more launch files in "node"'
		return None
	print 'Correct command parameters'

	return super(QProcessControl, cls).__new__(cls)
    
    def __init__(self, command):
        super(QProcessControl, self).__init__()
	print 'Init'

	self.command = command
	self.args = [command['pkg']]
	# In case we have a roslaunch command we extract all launch files from "node" and append them as command's arguments
	if command['command'] == 'roslaunch':
		for launchFile in command['node']:
			self.args.append(launchFile)
	else:
	# Else we have a rosrun and we can have only a single node we can run so we append directly "node"
		self.args.append(command['node'])
	self.status = False
	self.pid = 0
#	self.pidFilePath = 'qpc.pid'
	self.pidFilePath = command['pid']

        if os.path.isfile(self.pidFilePath):
		print 'Found \"' + self.pidFilePath + '\". Restoring connection to detached process'
		with open(self.pidFilePath) as f:
			self.pid = int(f.readline())	# TODO: this has to be long but damn it's so hard to parse long values in Python O_O
			print 'PID: ', self.pid
			self.status = True
	else:
		print 'Warning: No \"' + self.pidFilePath + '\" detected. If you have started the detached process, closed the UI and deleted this file, the application will be unable to restore its state and the external process will be orphaned!'
        self.initUI()
        
    def initUI(self):
        
	self.hbox = QHBoxLayout()

	self.qbtn = QPushButton('Start', self)
	self.qbtn.setCheckable(True)
	if self.status:
		self.qbtn.setChecked(True)
		self.qbtn.setText('Stop')
	self.qbtn.clicked.connect(self.toggleProcess)
	self.qbtn.resize(self.qbtn.sizeHint())
	self.hbox.addWidget(self.qbtn)

	self.setLayout(self.hbox)
	self.setGeometry(300, 300, 250, 150)
	self.setWindowTitle('QProcess controlled by a button')    
	self.show()

    def toggleProcess(self, val):
	if val:
		print 'Starting process'
		# Note: when roslaunch is terminated all processes spawned by it are also terminated
		# thus even if roscore has been started by the roslaunch process it will too be stopped :)
#		self.status, self.pid = QProcess.startDetached('roslaunch', ['lt', 'talker.launch'], '.') #(self.command, self.args, '.')
		self.status, self.pid = QProcess.startDetached(self.command['command'], self.args, '.') #(self.command, self.args, '.')
		if self.status:
			print 'PID: ', self.pid
			pidFile = open(self.pidFilePath, 'w')
			pidFile.write(str(self.pid))
			pidFile.close()
			self.qbtn.setText('Stop')			
		else:
			self.qbtn.setChecked(False)
			print 'Error: Failed to create process!'
	else:
		print 'Stopping process'
		if self.status:
			# kill takes a very short amount of time hence we can call it from inside the main thread without freezing the UI
			self.success = None
			if self.command['command'] == 'rosnode':
				self.success = subprocess.call(['rosnode', 'kill', 'talker'])
			else:
				if check_pid(self.pid):
					self.success = os.kill(self.pid, SIGINT)
				else:
					print 'Error: No process with PID ' + str(self.pid) + ' detected'
				        if os.path.isfile(self.pidFilePath):
						os.remove(self.pidFilePath)
#					self.qbtn.setChecked(True)
			# NOTE: using scripts (rosrun pkg_name script.py) and not launching ROS-confrom nodes creates
			# nodes like "talker_121314_12121414", which are impossible to distinguish without too much
			# fuss and make it really difficult to use 'rosnode kill' hence the requirement to start only
			# nodes that have a simple, distinguishable name so that rosnode kill can be used or use roslaunch
			# and launch files to give proper names

			# == 0 : for subprocess.call() return value | == None : for os.kill() return value (None -> kill was successful)
			if self.success == 0 or self.success == None:
				print 'Process stopped!'
				self.status = False
				self.pid = 0
                                if os.path.isfile(self.pidFilePath):
                                  os.remove(self.pidFilePath)
				self.qbtn.setText('Start')
			else:
				print 'Error: Failed to stop process!'
				self.qbtn.setChecked(True)

def main(): 
	app = QApplication(sys.argv)

	command = {
		'command' : 'roslaunch',
		'pkg' : 'lt',
		'node' :  ['talker.launch'],
		'pid' : 'talker.pid'
	}

	ex = QProcessControl(command)
   	if ex == None:
		sys.exit(1)

	sys.exit(app.exec_())


if __name__ == '__main__':
	main()
