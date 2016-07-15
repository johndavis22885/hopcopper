from pymongo import MongoClient
import pymongo

import os

# get database connection
def getConnection():
	# connect to local server
	client = MongoClient()	

	# connect to remote server
	#client = MongoClient("mongodb://mongodb0.example.net:27019") 

	# use a database named 'tradeindia'
	db = client.hotcopper
	return db

# check if data has key or not
# 		return data[key], if key is existed
#			return "", otherwise
def checkKey(data, key, second_key=""):
	# if key is existed
	if second_key == "" and key in data:
			temp = data[key].replace(",", "")
			temp = temp.replace("\"", " ")
			return temp
	elif second_key != "" and key in data and second_key in data[key]:
			temp = data[key][second_key].replace(",", "")
			temp = temp.replace("\"", " ")
			return temp
	# otherwise
	else:
			return ""

# read data from mongodb database and write them into a file with csv format.
def CSVFile():

	# get the absolute path of csv file
	# absolute_path = os.path.dirname(os.path.abspath(__file__))
	# path = absolute_path.split("tradeindia")[0] + "/tradeindia/" + "data.csv"

	# open csv file
	fp = open("data.csv", 'wb')

	# write header in csv file
	fp.write('"Post ID", "Forum","Forum Url","Tags","Tags Url","Poster Name","Poster Url",\
"Number of Posts","Message","Subject","Subject Url","Views","Rating","Date","Time","Price","Sentiment","Disclosure"\n')

	# get a connection of mongodb database
	db = getConnection()

	# get data group by search key and write data into csv file
	cursor = db.post.find()
	for document in cursor:

		line = '"%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s","%s"\n' % \
				(checkKey(document, 'post_id'), checkKey(document, 'forum'), checkKey(document, 'forum_url'),\
				checkKey(document, 'tags'), checkKey(document, 'tags'), checkKey(document, 'poster', 'name'), checkKey(document, 'poster', 'url'),\
				checkKey(document, 'poster', 'num'), checkKey(document, 'message'), checkKey(document, 'subject'), checkKey(document, 'subject_url'), \
				checkKey(document, 'views'), checkKey(document, 'rating'), checkKey(document, 'date'), checkKey(document, 'time'),\
				checkKey(document, 'price'), checkKey(document, 'sentiment'), checkKey(document, 'disclosure'))

		# encode data string with utf8
		fp.write(line.encode("utf8"))

	fp.close()

# read data from mongodb and write them into csv file
if __name__ == '__main__':

	CSVFile()

