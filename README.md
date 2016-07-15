# Hotcopper Scraper

Scrapers and data analysis code

1. Install packages

- Install Mongodb
	https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-14-04
- Install pymongo
	$ sudo pip install pymongo
- Install python scrapy
	$ sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev
	$ pip install Scrapy

2. Run program
- In hotcopper/, run the following command

	$ scrapy crawl hotcopper

3. Result

- The program saves data into 'post' table in 'hotcopper' database.
- data.csv and post.json in hotcopper/
