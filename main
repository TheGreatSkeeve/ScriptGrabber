from bs4 import BeautifulSoup
import pickle
import nltk
from nltk.util import ngrams
import requests


class script:
    def __init__(self, name, type):
        self.name = name
        self.type = type
        self.characters = []
        if self.type == "show":
            self.links = self.getLinks(name)
            self.getLines(self.links)
            self.num = 2
            while self.num > 1:
                self.prune()
            self.dumpData()
        elif self.type == "movie":
            self.url = 'https://imsdb.com/scripts/{}.html'.format(self.name.replace(" ","-"))
            self.getLines([self.url])
            self.num = 2
            while self.num > 1:
                self.prune()
            self.dumpData()
    def dumpData(self):
        for character in self.characters:
            name = character['name'].replace(" ", "").replace('"', "")
            self.makePKL(character['lines'], name)
            blank = ""
            for line in character['lines']:
                blank += line + "\n"
            f = open("{}.txt".format(name), "a", encoding='utf-8')
            f.write(blank)
            f.close()
    def list(self):
        data = []
        for character in self.characters:
            data.append(character['name'])
        return data
    def count(self):
        for character in self.characters:
            print("{} - {} lines".format(character['name'],len(character['lines'])))
    def addLine(self,name,line):
        found = 0
        for character in self.characters:
            if character['name'] == name:
                character['lines'].append(line)
                found = 1
        if found == 0:
            dict = {'name':name, 'lines': [line]}
            self.characters.append(dict)
    def getLinks(self,showName):
        showNameDash = showName.replace(" ","-")
        showNameTwenty = showName.replace(" ","%20")
        baseShow = 'https://imsdb.com/TV/{}.html'.format(showNameTwenty)
        raw = BeautifulSoup(requests.get(baseShow).content)("p")
        links = []
        for entry in raw:
            link = entry("a")[0]['href']
            link = link.replace('/TV Transcripts/{} - '.format(showName),"").replace(" ","-").replace("-Script","")
            full = "https://imsdb.com/transcripts/{}-".format(showNameDash) + link
            links.append(full)
        return links
    def getLines(self,links):
        for link in links:
            print(link)
            soup = BeautifulSoup(requests.get(link).content)
            try:
                for bold_tag in soup.find_all('b'):
                    if "[" in bold_tag.text or "[" in bold_tag.next_sibling.text or bold_tag.text == '':
                        pass
                    elif bold_tag.text.replace(" ", "").replace("'", "").replace("\r\n", "") == '':
                        pass
                    else:
                        name = bold_tag.text.replace("  ", "").replace("\r\n", "")
                        line = bold_tag.next_sibling.text.replace("  ", "").replace("\r\n", "")
                        if len(line) > 13:
                            self.addLine(name, line)
            except:
                pass
    def prune(self):
        pruned = 0
        for character in self.characters:
            if len(character['lines']) < 50:
                print("Removing character {}".format(character['name']))
                self.characters.remove(character)
                pruned += 1
        self.num = pruned
    def PInsert(self, dictionary, key, value):
        if key not in dictionary:
            dictionary[key] = {}
        if value not in dictionary[key]:
            dictionary[key][value] = 0
    # Dump a dictionary to a pkl
    def makePKL(self, data, name):
        self.quote_gen = {}
        for sentence in data:
            token = nltk.word_tokenize(sentence)
            for w1, w2, w3 in ngrams(token, 3, pad_left=True, pad_right=True):
                if (w1, w2) not in self.quote_gen:
                    self.quote_gen[(w1, w2)] = {}
                if w3 not in self.quote_gen[(w1, w2)]:
                    self.quote_gen[(w1, w2)][w3] = 0
                self.quote_gen[(w1, w2)][w3] += 1
        for w1_w2 in self.quote_gen:
            total_count = float(sum(self.quote_gen[w1_w2].values()))
            for w3 in self.quote_gen[w1_w2]:
                self.quote_gen[w1_w2][w3] /= total_count
        pickle.dump(self.quote_gen, open(name.lower() + '.pkl', 'wb'))

Futurama = script('Futurama')
