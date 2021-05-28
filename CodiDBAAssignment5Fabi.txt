import pprint	#To print pretty
import math		#To work with calculations
from collections import OrderedDict 	#To order the Dictionaries easier

#Here we create a simulation example of a list of client demands. We receive the Arriving ID, the asking Data rate and the Premium priority of each client
demand = {1:{'allocID':1,'bps':64000,'prem':1},
			2:{'allocID':2,'bps':8000,'prem':0},
			3:{'allocID':3,'bps':64000,'prem':1},
			4:{'allocID':4,'bps':64000,'prem':0},
			5:{'allocID':5,'bps':32000,'prem':0},
			6:{'allocID':6,'bps':96000,'prem':0},
			7:{'allocID':6,'bps':96000,'prem':1},
}

print("Client's demand:")
print('\n')
pprint.pprint(demand)

#Now we are sorting the client Dictionary by "prem" order. Highest "prem" value goes first.
#Source: https://realpython.com/python-ordereddict/#getting-started-with-pythons-ordereddict
orderedDemand = OrderedDict(sorted(demand.items(), key=lambda item: item[1]['prem'], reverse=True))

#Some variables to work with, as mentioned in the reading Document ITU-T
selDemand = dict()
usedMap = dict()
strng = "Impossible demand. Limit BW reached"
Rbe = 8000
Ra = 16000
Rf = 32000
Rm = Ra + Rf
limBW = 256000
j=1

#We iterate for item in the dictionary structure and do some code in it
for i in orderedDemand:
	aux = orderedDemand[i]

	#If the 'prem' section of the current item has value=1, more bandwidth is assigned in order to have priority in he uplink
	if aux['prem'] == 1:
		Rl = 48000
	else:
		Rl = 16000

	selDemand[j] = dict()
	
	bps = aux['bps']
	
	#A new dictionary is created in order to introduce new values regarding the old dictionary and the applying BW
	selDemand[j]['allocID'] = aux['allocID']
	selDemand[j]['bps'] = aux['bps']
	selDemand[j]['prem'] = aux['prem']
	
	#We decide the amount of BW that has to be assigned to each client looking at the priority indicated above and the amount of bandwidth the client is asking for, so more or less BW can be assigned and no excessive BW is consumed
	if bps <= Rf:
		selBw = Rf
		selDemand[j]['selBW'] = selBW
	elif bps <= Rm:
		selBW = Rm
		selDemand[j]['selBW'] = selBW
	elif bps <= Rm + Rl:
		selBW = Rm + Rl
		if selBW == limBW:
			selDemand[j]['selBW'] = strng

		#if the bandwidth that should go to the client excesses or almost arrives to the limit BW value, a message is sent in order to acknowledge why is not possible to do that and a new Best-effort BW is assigned in order to have some BW until more data can be given
			aux = min(Rm,Rl) - Ra
			selDemand[j]['newSelBW'] = Rl + aux
		else:
			selDemand[j]['selBW'] = selBW
	else:
		selDemand[j]['selBW'] = strng
		selDemand[j]['newSelBW'] = Rl + min(Rm,Rl) - Ra

	j = j + 1

#New Dictionary with BW assignments is shown
print('\n')
print("Clients ordered by premium priority and BW assigned:")
print('\n')
pprint.pprint(selDemand)

j = 1
startBit = 99999
stopBit = 0

#When all clients are iterated, a second iterations is done in order to assign the corresponding Grant Map frames for each possible client 
for i in orderedDemand:
	aux = orderedDemand[i]
	aux2 = selDemand[i]

	#If the client is a premium one, more fps are assignet to give in order to have better resolution so more BW is needed to as the priority indicates
	if aux['prem'] == 1:
		fps = 60
	else:
		fps = 30

	usedMap[j] = dict()

	#A new Dictionary is created with new elements per item regarding the start and stop frames
	usedMap[j]['allocID'] = aux['allocID']
	selBW = aux2['selBW']
    
	if selBW != strng:
		usedMap[j]['usedBw'] = selBW
	else:
		usedMap[j]['usedBw'] = aux2['newSelBW']

	#For each consecutive client, each client starts receiveng immediately the bit after the previous one
	startBit = stopBit + 1
	stopBit = fps * usedMap[j]['usedBw'] + startBit
	usedMap[j]['startBit'] = startBit
	usedMap[j]['stopBit'] = stopBit
	usedMap[j]['prem'] = aux['prem']

	j = j + 1

#Finnally the Grant Map is shown with the indexed BW for each client and the corresponding time for each one amount other information
print('\n')
print("Grant Map created with start and stop frames for each client:")
print('\n')
pprint.pprint(usedMap)