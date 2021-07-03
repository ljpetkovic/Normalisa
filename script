import re
import os
import csv
import spacy
from spacy.tokenizer import Tokenizer
from spacy.lang.fr import French
import fileinput
from lxml import etree
from unidecode import unidecode


"""
A FAIRE :
- ajout conjugaisons auxiliaires
- ajout variantes noms propres
"""

dict_AF = {} # dict ancien français général (noms, adv, adj, pronoms) vs forme moderne
dict_AF_verbs = {} # dict ancien français verbes avec leurs conjugaisons vs infinitif moderne
dict_AF_names = {} # dict ancien français noms propres vs forme moderne
with open("LGeRM-LexiqueMorphologique-MODE-1.0.0.xml", encoding="latin-1") as LGeRM:
    formset = ""
    for line in LGeRM:
        if "<formSet" in line:
            formset += line
            continue
        if "</formSet" in line:
            formset += line
            match_words = re.findall(r"<orthography>([^<]+)", formset, re.S)
            match_pos = re.findall(r"<grammaticalCategory>([^<]+)", formset, re.S)
            if match_words:
                if "nom propre" in match_pos and "nom_propre" not in match_words and "nom_propres" not in match_words:
                    inflections = match_words[1:]
                    lemma = match_words[0]
                    for inflection in inflections:
                        dict_AF_names[inflection] = lemma
                        continue
                elif "verbe" in match_pos:
                    inflections = match_words[1:]
                    lemma = match_words[0]
                    for inflection in inflections:
                        dict_AF_verbs[inflection] = lemma
                        continue
                else:
                    if "nom_propre" not in match_words and "nom_propres" not in match_words:
                        inflections = match_words[1:]
                        lemma = match_words[0]
                        for inflection in inflections:
                            dict_AF[inflection] = lemma
            formset = ""
        if formset != "":
            formset += line
            
dict_FM = {}
with open("Morphalou-1.0.1.xml", encoding="latin-1") as Morphalou:
    formset = ""
    for line in Morphalou:
        if "<lexicalEntry" in line:
            formset += line
            continue
        if "</lexicalEntry" in line:
            formset += line
            match_words = re.findall(r'<inflection orthography="([^"]+)', formset, re.S)
            match_lemma = re.findall(r'<lexicalEntry lemma="([^"]+)', formset, re.S)
            if match_words:
                lemma = match_lemma[0]
                for inflection in match_words:
                    dict_FM[inflection] = lemma
            formset = ""
        if formset != "":
            formset += line

latin_exceptions = []
with open("Liste_latin.txt", encoding="latin-1") as text:
    text = text.read()
    text = text.replace("\n", " ")
    tokens = text.split()
    for token in tokens:
        latin_exceptions.append(token)

def tokenized_corpus(path):
    filenames = os.listdir(path)
    corpus_tokens = []
    tokens_found = set()
    ref_token = set()
    for filename in filenames:
        file_path = os.path.join(path,filename)
        with open(file_path) as file_object:
            try:
                doc = file_object.read()
                print("{} est en cours...".format(filename), flush=False)
            except:
                print("{} n'a pas pu être traité.".format(filename), flush=False)
                continue
        doc = doc.replace("\n", " ")
        doc = re.sub(r"<[^>]+>", " ", doc)
        doc = re.findall(r'\w+|&', doc)
        for token in doc:
            #if len(token) == 1:
                #continue
            if re.search(r"\d", token):
                continue
            if token in tokens_found:
                continue
            if token in ref_token:
               continue
            if re.search(r"\w|&", token):
                ref_token.add(token)
                tokens_found.add(token.lower())
                corpus_tokens.append(token.lower())
    return sorted(corpus_tokens)
   

# Construire un dictionnaire de conversion à partir de règles :

def building_new_dict(words_list):
    new_dict = {}
    for word in words_list:
        new_word = ""
        word_dict = word
        if word_dict in dict_FM:
            continue
        if word_dict in latin_exceptions:
            continue
        if word_dict in new_dict:
            continue
        if word_dict in dict_AF_names and word_dict not in dict_AF:
            new_word = dict_AF_names[word_dict]
            if new_word != word_dict:
                new_dict[word_dict] = new_word
            continue
        else:
            new_word = word
        new_word = re.sub(r"ſ", r"s", new_word)
        new_word = re.sub(r"&", r"et", new_word)
        new_word = re.sub(r"evs", r"eus", new_word)
        new_word = re.sub(r"esv", r"êv", new_word)
        if re.findall(r"y$", new_word):
            if new_word in dict_AF_verbs and new_word != "ay" and dict_AF_verbs[new_word] != "ouïr":
                new_word = re.sub(r"y$", r"is", new_word)
            else:
                new_word = re.sub(r"y$", r"i", new_word)
        new_word = re.sub(r"tost$", r"tôt", new_word)
        new_word = re.sub(r"ist$", r"ît", new_word)
        new_word = re.sub(r"ust$", r"ût", new_word)
        new_word = re.sub(r"ûst$", r"ût", new_word)
        new_word = re.sub(r"ast$", r"ât", new_word)
        new_word = re.sub(r"oist", r"oît", new_word)
        new_word = re.sub(r"õ", r"on", new_word)# 
        new_word = re.sub(r"ã", r"an", new_word)# 
        new_word = re.sub(r"ẽ", r"en", new_word)#
        new_word = re.sub(r"qv", r"qu", new_word)
        new_word = re.sub(r"sç", r"s", new_word)
        new_word = re.sub(r"oust", r"oût", new_word)
        new_word = re.sub(r"ee$", r"ée", new_word)
        new_word = re.sub(r"ees$", r"ées", new_word)
        new_word = re.sub(r"^dez$", r"des", new_word)
        new_word = re.sub(r"œ", r"oe", new_word)
        trans_word = re.sub(r"u", r"v", new_word)#
        if trans_word in dict_FM:
            new_word = trans_word
        trans_word = re.sub(r"v", r"u", new_word)#
        if trans_word in dict_FM:
            new_word = trans_word
        trans_word = re.sub(r"uu", r"uv", new_word)#
        if trans_word in dict_FM:
            new_word = trans_word
        trans_word = re.sub(r"est", r"êt", new_word)#
        if trans_word in dict_FM:
            new_word = trans_word
        if re.search(r"oiss", new_word):
            new_word = re.sub(r"oiss", r"aiss", new_word)
        if re.search(r"ans$", new_word) and new_word not in dict_FM:
            new_word = re.sub(r"ans", r"ants", new_word)
        if re.search(r"lx$", new_word):
            new_word = re.sub(r"lx", r"x", new_word)
        if re.search(r"est$", new_word) and new_word != "est":
            new_word = re.sub(r"est$", r"et", new_word)
        
        #cas particuliers :

        new_word = re.sub(r"^ie$", r"je", new_word)
        new_word = re.sub(r"^i$", r"j", new_word)
        new_word = re.sub(r"^creu$", r"cru", new_word)
        new_word = re.sub(r"^veuë$", r"vue", new_word)
        new_word = re.sub(r"^vostre$", r"votre", new_word)
        new_word = re.sub(r"^dailleurs$", r"d'ailleurs", new_word)
        new_word = re.sub(r"^ceque$", r"ce que", new_word)
        new_word = re.sub(r"^aussibien$", r"aussi bien", new_word)
        new_word = re.sub(r"^cest$", r"c'est", new_word)
        new_word = re.sub(r"^aucunes$", r"aucune", new_word)
        new_word = re.sub(r"^aucuns$", r"aucun", new_word)
        new_word = re.sub(r"^chacuns$", r"chacun", new_word)
        new_word = re.sub(r"^chacuns$", r"chacun", new_word)
        new_word = re.sub(r"^apres$", r"après", new_word)
        new_word = re.sub(r"^aprés$", r"après", new_word)
        new_word = re.sub(r"^auprés$", r"auprès", new_word)
        new_word = re.sub(r"^ayeux$", r"aïeux", new_word)
        new_word = re.sub(r"^quon$", r"qu'on", new_word)
        new_word = re.sub(r"^quil$", r"qu'il", new_word)
        new_word = re.sub(r"^encor$", r"encore", new_word)
        new_word = re.sub(r"^estre$", r"être", new_word)
        new_word = re.sub(r"^parceque$", r"parce que", new_word)
        new_word = re.sub(r"^abatu$", r"abattu", new_word)
        new_word = re.sub(r"^abatue$", r"abattue", new_word)
        new_word = re.sub(r"^pere$", r"père", new_word)

        if new_word in dict_FM:
            new_dict[word_dict] = new_word
            continue

        #traiter en particulier les adj et les noms:

        if new_word in dict_AF:
            if re.findall(r"[a-zâéîïûüëèêç]{1}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{1}$", dict_AF[new_word]) and len(new_word) == len(dict_AF[new_word]):
                new_word = dict_AF[new_word]
                new_dict[word_dict] = new_word
                continue
            if re.findall(r"[a-zâéîïûüëèêç]{2}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{2}$", dict_AF[new_word]):
                new_word = dict_AF[new_word]
                new_dict[word_dict] = new_word
                continue
            else:
                try:
                    if re.search(r"ens$", new_word) and re.search(r"ent$", dict_AF[new_word]):
                        new_word = re.sub(r"t$", r"ts", dict_AF[new_word])
                    if re.search(r"ez$", new_word) and new_word not in dict_AF_verbs:
                        new_word = re.sub(r"ez", r"és", new_word)
                    if re.search(r"tems$", new_word):
                        new_word = re.sub(r"ms$", r"mps", new_word)
                    if re.search(r"ü$", new_word):
                        if re.search(r"ü", dict_AF[new_word]) is None:
                            new_word = re.sub(r"ü", r"u", new_word)
                    if re.search(r"e$", new_word) and re.search(r"é$", dict_AF[new_word]) and new_word not in dict_AF_verbs and len(new_word) > 2:
                        new_word = dict_AF[new_word]
                    if re.search(r"a$", new_word) and re.search(r"à$", dict_AF[new_word]) and len(new_word) > 2:
                        new_word = dict_AF[new_word]
                    if re.findall(r"uës?$", new_word):
                        new_word = re.sub(r"ë", r"e", new_word)
                    if re.findall(r"ües?$", new_word):
                        new_word = re.sub(r"ü", r"u", new_word)
                    if re.search(r"s$", new_word):
                        sing_word = re.sub(r"s$", r"", new_word)
                        if re.findall(r"[a-zâéîïûüëèêç]{2}$", sing_word) == re.findall(r"[a-zâéîïûüëèêç]{2}$", dict_AF[new_word]):
                            new_word = re.sub(r"$", r"s", dict_AF[new_word])
                    if re.search(r"eaux$", new_word):
                        if re.findall(r"eau$", dict_AF[new_word]):
                            new_word = re.sub(r"$", r"x", dict_AF[new_word])
                    if re.search(r"[dlnprstueé]e$", new_word):
                        masc_word = re.sub(r"e$", r"", new_word)
                        if re.findall(r"[a-zâéîïûüëèêç]{2}$", masc_word) == re.findall(r"[a-zâéîïûüëèêç]{2}$", dict_AF[new_word]):
                            new_word = re.sub(r"$", r"e", dict_AF[new_word])
                    if re.search(r"[dlnprstueé]es$", new_word):
                        masc_word = re.sub(r"es$", r"", new_word)
                        if re.findall(r"[a-zâéîïûüëèêç]{2}$", masc_word) == re.findall(r"[a-zâéîïûüëèêç]{2}$", dict_AF[new_word]):
                            new_word = re.sub(r"$", r"es", dict_AF[new_word])
                    if re.findall(r"ere$", new_word) and re.findall(r"er$", dict_AF[new_word]) and len(new_word) == len(dict_AF[new_word]) + 1:
                        new_word = re.sub(r"er$", r"ère", dict_AF[new_word])
                    if re.findall(r"eres$", new_word) and re.findall(r"er$", dict_AF[new_word]) and len(new_word) == len(dict_AF[new_word]) + 2:
                        new_word = re.sub(r"er$", r"ères", dict_AF[new_word])
                    if re.findall(r"euse$", new_word) and re.findall(r"eur$", dict_AF[new_word]):
                        new_word = re.sub(r"eur$", r"euse", dict_AF[new_word])
                    if re.findall(r"euse$", new_word) and re.findall(r"eux$", dict_AF[new_word]):
                        new_word = re.sub(r"eux$", r"euse", dict_AF[new_word])
                    if re.findall(r"euses$", new_word) and re.findall(r"eur$", dict_AF[new_word]):
                        new_word = re.sub(r"eur$", r"euses", dict_AF[new_word])
                    if re.findall(r"euses$", new_word) and re.findall(r"eux$", dict_AF[new_word]):
                        new_word = re.sub(r"eux$", r"euses", dict_AF[new_word])
                    if re.findall(r"ete$", new_word) and re.findall(r"et$", dict_AF[new_word]) and len(new_word) == len(dict_AF[new_word]) + 1:
                        new_word = re.sub(r"et$", r"ète", dict_AF[new_word])
                    if re.findall(r"etes$", new_word) and re.findall(r"et$", dict_AF[new_word]) and len(new_word) == len(dict_AF[new_word]) + 2:
                        new_word = re.sub(r"et$", r"ètes", dict_AF[new_word])
                    if re.findall(r"rice$", new_word) and re.findall(r"eur$", dict_AF[new_word]):
                        new_word = re.sub(r"eur$", r"rice", dict_AF[new_word])
                    if re.findall(r"rices$", new_word) and re.findall(r"eur$", dict_AF[new_word]):
                        new_word = re.sub(r"eur$", r"rices", dict_AF[new_word])
                    if re.findall(r"ive$", new_word) and re.findall(r"if$", dict_AF[new_word]):
                        new_word = re.sub(r"if$", r"ive", dict_AF[new_word])
                    if re.findall(r"ives$", new_word) and re.findall(r"if$", dict_AF[new_word]):
                        new_word = re.sub(r"if$", r"ives", dict_AF[new_word])
                    if re.findall(r"ienne$", new_word) and re.findall(r"ien$", dict_AF[new_word]):
                        new_word = re.sub(r"ien$", r"ienne", dict_AF[new_word])
                    if re.findall(r"iennes$", new_word) and re.findall(r"ien$", dict_AF[new_word]):
                        new_word = re.sub(r"ien$", r"iennes", dict_AF[new_word])
                    if re.findall(r"ouse$", new_word) and re.findall(r"oux", dict_AF[new_word]):
                        new_word = re.sub(r"oux$", r"ouse", dict_AF[new_word])
                    if re.findall(r"ouses$", new_word) and re.findall(r"oux", dict_AF[new_word]):
                        new_word = re.sub(r"oux$", r"ouses", dict_AF[new_word])
                    if re.findall(r"elle$", new_word) and re.findall(r"el", dict_AF[new_word]):
                        new_word = re.sub(r"el$", r"elle", dict_AF[new_word])
                    if re.findall(r"elles$", new_word) and re.findall(r"el", dict_AF[new_word]):
                        new_word = re.sub(r"el$", r"elles", dict_AF[new_word])
                except:
                    continue

        #traiter les formes verbales :

        if new_word in dict_AF_verbs:
            try:
                new_word = re.sub(r"ye$", r"ie", new_word)
                new_word = re.sub(r"yent$", r"ient", new_word)
                if "ï" in new_word:
                    if re.search(r"ïent$", new_word) or re.search(r"ïe$", new_word) or re.search(r"ïer[a-z]+$", new_word) or re.search(r"ïr$", new_word) or re.search(r"ir$", dict_AF_verbs[new_word]):
                        new_word = re.sub(r"ï", r"i", new_word)
                    else:
                        new_word = re.sub(r"ï", r"y", new_word)
                new_word = re.sub(r"[uû]ë", r"ue", new_word)
                new_word = re.sub(r"oü", r"ou", new_word)
                new_word = re.sub(r"ûë", r"ue", new_word)
                new_word = re.sub(r"ûs$", r"us", new_word)
                new_word = re.sub(r"û$", r"u", new_word)
                new_word = re.sub(r"ûe$", r"ue", new_word)
                new_word = re.sub(r"ûes$", r"ues", new_word)
                new_word = re.sub(r"oust", r"oût", new_word)
                new_word = re.sub(r"aist", r"aît", new_word)
                new_word = re.sub(r"ast$", r"ât", new_word)
                if re.search(r"ois$", new_word):
                    if dict_AF_verbs[new_word] == "avoir" or dict_AF_verbs[new_word] == "savoir":
                        new_word = re.sub(r"ois$", r"ais", new_word)
                    if new_word in dict_AF_verbs and re.search(r'oire?$', dict_AF_verbs[new_word]) is None:
                        new_word = re.sub(r"ois$", r"ais", new_word)
                if re.search(r"oit$", new_word):
                    if dict_AF_verbs[new_word] == "avoir" or dict_AF_verbs[new_word] == "savoir":
                        new_word = re.sub(r"oit$", r"ait", new_word)
                    if re.search(r'oire?$', dict_AF_verbs[new_word]) is None:
                        new_word = re.sub(r"oit$", r"ait", new_word)
                if re.search(r"oient$", new_word):
                    if dict_AF_verbs[new_word] == "avoir" or dict_AF_verbs[new_word] == "savoir":
                        new_word = re.sub(r"oient$", r"aient", new_word)
                    if re.search(r'oire?$', dict_AF_verbs[new_word]) is None:
                        new_word = re.sub(r"oient", r"aient", new_word)
                if re.search(r"evoit$", new_word) or re.search(r"evois$", new_word) or re.search(r"evoient$", new_word):
                    new_word = re.sub(r"oi", r"ai", new_word)
                if re.search(r"oît$", new_word):
                    new_word = re.sub(r"oî", r"aî", new_word)
                if re.findall(r"as[tm]", new_word):
                    if "â" in dict_AF_verbs[new_word]:
                        new_word = re.sub(r"as", r"â", new_word)
                if new_word in dict_FM:
                    new_dict[word_dict] = new_word
                    continue
            except:
                continue
            for a, b in dict_FM.items():
                try:
                    if dict_AF_verbs[new_word] == b and new_word not in dict_FM:
                        if re.findall(r"[eé]e$", new_word) and re.findall(r"ée$", a):
                            new_word = a
                        if re.findall(r"[eé]es$", new_word) and re.findall(r"ées$", a):
                            new_word = a
                        if re.findall(r"ue$", new_word) and re.findall(r"ue$", a):
                            new_word = a
                        if re.findall(r"ant$", new_word) and re.findall(r"ant$", a):
                            new_word = a
                        if re.findall(r"ants$", new_word) and re.findall(r"ants$", a):
                            new_word = a
                        if re.findall(r"ante$", new_word) and re.findall(r"ante$", a):
                            new_word = a
                        if re.findall(r"antes$", new_word) and re.findall(r"antes$", a):
                            new_word = a
                        if dict_AF_verbs[new_word] == "avoir":
                            if re.findall(r"[a-zâéîïûüëèêç]{2}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{2}$", a) and len(new_word) == len(a):
                                new_word = a
                                new_dict[word_dict] = new_word
                                continue
                        if len(new_word) <= len(b):
                            if re.findall(r"[a-zâéîïûüëèêç]{3}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{3}$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                            elif re.findall(r"[eé][rnvt]e$", new_word) and re.findall(r"è[rnvt]e$", a):
                                new_word = a
                            elif re.findall(r"[eé][rnvt]es$", new_word) and re.findall(r"è[rnvt]es$", a):
                                new_word = a
                            elif re.findall(r"[eé][rnvt]a$", new_word) and re.findall(r"è[rnvt]a$", a):
                                new_word = a
                            elif re.findall(r"[eé][rnvt]as$", new_word) and re.findall(r"è[rnvt]as$", a):
                                new_word = a
                            elif re.findall(r"[eé]rent$", new_word) and re.findall(r"èrent$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                        elif len(new_word) <= len(b) + 2:
                            if re.findall(r"[a-zâéîïûüëèêç]{4}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{4}$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                            elif re.findall(r"[eé]rent$", new_word) and re.findall(r"èrent$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                        elif len(new_word) == len(b) + 3:
                            if re.findall(r"[a-zâéîïûüëèêç]{6}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{6}$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                            elif re.findall(r"[eé]rent$", new_word) and re.findall(r"èrent$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                        elif len(new_word) >= len(b) + 4:
                            if re.findall(r"[a-zâéîïûüëèêç]{7}$", new_word) == re.findall(r"[a-zâéîïûüëèêç]{7}$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                            elif re.findall(r"[eé]rent$", new_word) and re.findall(r"èrent$", a) and len(new_word) - 1 <= len(a) <= len(new_word) + 1:
                                new_word = a
                        if re.findall(r"ois$", new_word):
                            if re.findall(r"ais$", a) and len(new_word) == len(a):
                                new_word = re.sub(r"ois$",r"ais", new_word)
                        if re.findall(r"oit$", new_word):
                            if re.findall(r"ait$", a) and len(new_word) == len(a):
                                new_word = re.sub(r"oit$",r"ait", new_word)
                        if re.findall(r"oient$", new_word):
                            if re.findall(r"aient$", a) and len(new_word) == len(a):
                                new_word = re.sub(r"oient$",r"aient", new_word)
                    else:
                        if new_word in dict_FM:
                            new_dict[word_dict] = new_word
                        continue
                except:
                    continue

        if new_word in dict_FM:
            new_dict[word_dict] = new_word
        else:#
            if re.search(r"ui", new_word):
                trans_word = re.sub(r"ui", r"vi", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"uiu", new_word):
                trans_word = re.sub(r"uiu", r"viv", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"iu", new_word):
                trans_word = re.sub(r"iu", r"iv", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"eu", new_word):
                trans_word = re.sub(r"eu", r"ev", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"uu", new_word):
                trans_word = re.sub(r"uu", r"uv", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"uu", new_word):
                trans_word = re.sub(r"uu", r"vu", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"eu$", new_word):
                trans_word = re.sub(r"eu", r"u", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if re.search(r"[âéîïûüëèê]", new_word):
                trans_word = re.sub(r"â", r"a", new_word) or re.sub(r"[éèêë]", r"e", new_word) or re.sub(r"[îï]", r"i", new_word) or re.sub(r"[ûü]", r"u", new_word)
                if trans_word in dict_FM:
                    new_word = trans_word
            if new_word in dict_AF_names:
                new_word = dict_AF_names[new_word]
            #if new_word != word_dict:
            if new_word in dict_FM:
                new_dict[word_dict] = new_word
            else:
                continue
            
    return(new_dict)

if __name__ == "__main__":
    path = ""
    words_list = tokenized_corpus(path)
    results = building_new_dict(words_list)
    print("dict done")
    with open("dict-modernisation-Holbach.text", "w") as doc:
        for i, y in results.items():
                print(i, " : ", y, file=doc)
   
