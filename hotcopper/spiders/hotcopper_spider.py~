import scrapy

from hotcopper.db_manager import *

class HotcopperSpider(scrapy.Spider):
	name = "hotcopper"
	allowed_domains = ["hotcopper.com.au"]

	domain = "http://hotcopper.com.au/"

	def __init__(self):
		self.user = "papina"
		self.password = "superman"
		
		self.db = getConnection()
		
		# remove all data in database
		self.db.post.remove({})
		
	def start_requests(self):
		
		'''request = scrapy.FormRequest("http://hotcopper.com.au/login/login", headers=self.headers,
						formdata={'login': self.user, 'password': self.password, 'tos':'Y', \
						'cookie_check':'1', 'redirect':'http://hotcopper.com.au', '_xfToken':''},
						callback=self.parse_posts)

		return [request]'''
			
		request = scrapy.Request(self.domain + "postview/page-1", callback=self.parse_posts)
		request.meta["page"] = 1
		return [request]

	# get list of all posts
	def parse_posts(self, response):
		# get html nodes of posts
		posts = response.xpath("//li[contains(@class, 'post-item')]")
		
		# get info of a post from post list page
		for post in posts:
			res_posts = dict()
			
			targets = ['forum', 'tags', 'author']
			
			# crawl data (forum, tags, subject, author)
			
			for target in targets:
				if target == 'author':
					res_posts['poster'] = dict()
					res_posts['poster']['name'] = self.validate(post.xpath(".//div[contains(@class, '%s')]//a/text()" % target))
					res_posts['poster']['url'] = self.validate(post.xpath(".//div[contains(@class, '%s')]//a/@href" % target))
					if res_posts['poster']['url'] != "":
						res_posts['poster']['url'] = self.domain + res_posts['poster']['url']
						
				else:
					res_posts[target] = self.validate(post.xpath(".//div[contains(@class, '%s')]//a/text()" % target))
					res_posts[target + '_url'] = self.validate(post.xpath(".//div[contains(@class, '%s')]//a/@href" % target))
					if res_posts[target + '_url'] != "":
						res_posts[target + '_url'] = self.domain + res_posts[target + '_url']
			
				if target == 'forum' and res_posts['forum'] == "Sponsored":
					continue
				
			# get subject
			res_posts['subject'] = self.validate(post.xpath(".//div[contains(@class, 'subject')]//h3//a/text()"))
			res_posts['subject_url'] = self.domain + self.validate(post.xpath(".//div[contains(@class, 'subject')]//h3//a/@href"))
			
			# get views
			res_posts['views'] = self.validate(post.xpath(".//div[contains(@class, 'stats')]/text()"))
			
			# get rating
			res_posts['rating'] = ""
			tp_rating = post.xpath(".//div[contains(@class, 'rating')]/text()")
			if len(tp_rating) > 1:
				res_posts['rating'] = tp_rating[1].extract().strip().encode("utf8")
			
			# get date
			# res_posts['date'] = self.validate(post.xpath(".//div[contains(@class, 'time')]/@title"))
			
			# get post id
			tp_post_id = res_posts['subject_url'].split("post_id=")
			if len(tp_post_id) > 1:
				res_posts['post_id'] = tp_post_id[1]
					
			# make request for scraping post detail info
			request = scrapy.Request(res_posts['subject_url'], callback=self.parse_post_detail, dont_filter=True)
			request.meta['post'] = res_posts
			yield request			
			
		# make a request for the next page
		if 'page' not in response.meta:
			current_page = 1
		else:
			current_page = response.meta["page"]
			
		next = self.validate(response.xpath("//div[@class='PageNav']//a[contains(@class, 'page-disabled')]/text()"))
		
		# if there is no more next page, exit spider
		if next == "Next":
			return
		else: # otherwise, make request for the next page
			next_url = "%s/page-%d" % ('/'.join(response.url.split("/")[:-1]), current_page + 1)
			request = scrapy.Request(next_url, callback=self.parse_posts, dont_filter=True)
			request.meta['page'] = current_page + 1
			yield request
		
	# get data from post detail page
	def parse_post_detail(self, response):
		post = response.meta['post']
		
		id_name = "post-%s" % post['post_id']
		post_node = response.xpath("//li[@id='%s']" % id_name)
		
		if len(post_node) > 0:
			post_node = post_node[0]
			
			# get post message
			tp_message = post_node.xpath(".//blockquote[contains(@class, 'messageText')]/text()")
			post['message'] = ""
			for token in tp_message:
				post['message'] += token.extract().strip().encode("utf8")
			post['message'] =post['message'].strip()
			
			# get the number of posts that the user post.
			post['poster']['num'] = self.validate(post_node.xpath(".//div[@class='user-wrap']//span[@class='nb-posts']/text()")).split(" ")[0]
			
			# get thread meta data
			node = post_node.xpath(".//div[@class='message-user-metadata']//dt")
			
			for temp in node:
				label = self.validate(temp.xpath(".//label/text()"))
				
				if label == "Date:":
					post["date"] = self.validate(temp.xpath("./following-sibling::dd[1]/text()"))
				elif label == "Time:":
					post["time"] = self.validate(temp.xpath("./following-sibling::dd[1]/text()"))
				elif label == "Post #:":
					post["post_id"] = self.validate(temp.xpath("./following-sibling::dd[1]//a/text()"))
				
			node = post_node.xpath(".//div[contains(@class, 'message-user-metadata-sentiment')]//dt")

			if len(node) > 1:
				for temp in node[1:]:
					label = self.validate(temp.xpath(".//label/text()"))
					
					if label == "Price at posting:":
						post["price"] = self.validate(temp.xpath("./following-sibling::dd[1]/text()"))
					elif label == "Sentiment:":
						post["sentiment"] = self.validate(temp.xpath("./following-sibling::dd[1]/text()"))
					elif label == "Disclosure:":
						post["disclosure"] = self.validate(temp.xpath("./following-sibling::dd[1]/text()"))
						
		# save all data of one post into database.
		self.db.post.insert(post)

			
	def validate(self, node):
		if len(node) > 0:
			temp = node[0].extract().strip().encode("utf8")
			return temp
		else:	
			return ""
			
