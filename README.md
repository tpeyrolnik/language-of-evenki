# language-of-evenki
Данная программа предназначена для автоматического определения эвенкийского языка в текстах сообщений и постах (можно использовать другие агглютинативные языки России)

Программа работает с файлами json. В неё можно помещать данные из ВКонтакте или других соц.сетей ([тут](https://github.com/hse-ling-python/seminars/tree/19-20-archive/different_api) про то, как можно такие данные получать), или взять готовые, например, [отсюда](http://web-corpora.net/wsgi3/minorlangs/download). Главное, чтобы нужный текст был под ключом “text”

Программа использует [корпус эвенкийских морфем](https://github.com/tpeyrolnik/language-of-evenki/blob/main/morpheme_dictionary.csv), изначально существующий в латинице, поэтому в программу встроен транслитератор с эвенкийского ([здесь](https://github.com/lalsnivts/evenki_ocr_texts/blob/master/src/transliteration.py) то, на чём он основан). Таким образом в итоге корпус существует в виде списка с морфемами, написанными на кириллице. 

Основные функции: 
`dict_flat (my_gotov)` 

принимает в качестве аргумента копию json: 

```
chats = data
        gotov = chats.copy()
```

создаёт вторую копию, которую можно будет менять:

```
my_kotov = my_gotov.copy()
```

с помощью рекурсии ищет ключ ‘text’ и токенизирует текст сообщения/поста, оставляя только буквы (потому что язык определяется по словам, а слова состоят из букв), вызывает вторую функцию `detect_lang` и по окончании её работы изменяет `my_kotov`, добавляя в него новый ключ : 

```
my_kotov = my_gotov.copy()
        for key, value in my_gotov.items():
            if key == 'text':
                text_list = my_gotov[key].split(' ')
                my_kotov["lang"] = detect_lang(text_list)
            if type(my_gotov[key]) == dict:
                my_kotov[key] = dict_flat(my_gotov[key])
            elif type(my_gotov[key]) == list:
                processed_cats = []
                for el in my_gotov[key]:
                    processed_cats.append(dict_flat(el))
                my_kotov[key] = processed_cats 
```
            
возвращает `my_kotov`:
 
```
return my_kotov
```


`detect_lang(text_list)`

принимает в качестве аргумента токенизированный текст, с помощью цикла  ищет каждый токен в словаре эвенкийских морфем:
    
```
count = 0
   for chat_word in text_list:
        for word in updated_corpus:
            if word in chat_word:
                count += 1
```

считает количество слов, найденных в корпусе. если их больше половины, язык сообщения -- эвенкийский. если меньше -- по умолчанию русский:

```
if count > len(text_list) / 2:
        lang = 'even'
   else:
        lang = 'rus'
   return lang
```

в результате в копии изначального файла json появляется новый ключ `lang` с соответствующим языком. данные в новом файле: 

```
with open('new.json', 'w', encoding='utf-8') as failik:
        json.dump(dict_flat(gotov), failik)
```
 


