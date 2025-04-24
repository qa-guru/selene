# Лекция: Автоматизация тестирования с помощью Selene в Python

## Оглавление

1. [Введение](#Введение)
2. [Установка Selene](#1-установка-selene)
3. [Знакомство с Selene](#2-знакомство-с-selene)
4. [Конфигурация браузера](#3-конфигурация-браузера-selene)
5. [Работа с элементами](#4-поиск-элементов-и-коллекций-элементов)
6. [Локаторы: CSS vs XPath](#5-css-локаторы)
7. [Проверки и условия](#5-взаимодействие-с-элементами-и-коллекциями-элементов)
9. [Ресурсы](#ресурсы)

## Введение

Selene - это лаконичная и мощная библиотека для написания браузерных UI-тестов на Python. Она была создана как
Pythonic-порт популярного проекта Selenide из мира Java. Selene помогает разработчикам писать читаемые и сопровождаемые
тесты, которые "говорят" на обычном английском языке, что облегчает их понимание и совместное использование в командах.

Основная сила Selene - ориентированный на пользователя API, который абстрагирует всю сложность работы с Selenium
WebDriver. Тесты могут быть написаны с использованием простого, выразительного синтаксиса и поддерживают такие функции,
как элементы с ленивой оценкой, механизмы автоматического повторения для более умного неявного ожидания при
Ajax-подобной загрузке. Встроенная поддержка PageObject позволяет повторно использовать веб-элементы с помощью виджетов.

## 1. Установка Selene

Первое, что нам нужно сделать, это установить **selene** и **pytest**. Мы будем использовать следующие команды:

Первая

- **Флаг `--pre`**  
  Selene 2.0 находится в стадии предрелиза. Для установки актуальной версии используйте:

```bash
pip install selene --pre
```

Вторая

```bash
pip install pytest
```

Это установит последнюю версию библиотеки **selene** и тестовый фреймворк **pytest**.
После установки мы убедимся, что библиотека корректно импортируется в наш проект.

Это можно сделать, выполнив в терминале следующую команду:

```bash
pip list
```

В консоли терминала будет примерно такой вывод:

```text
Package           Version
----------------- -----------
...
pytest            8.3.3
selene            2.0.0rc9
...
```

Главное здесь, что мажорная версия `selene` равна `2`

Если бы мы выполнили команду установки библиотеки `selene` без флага `--pre`, тогда бы установилась старая неактуальная
версия

## 2. Знакомство с Selene

Selene — это обертка (или, как еще говорят "враппер") над Selenium, и она была создана для простоты работы с элементами
на веб-страницах.

Напишем два теста на `selenium` и `selene`, а потом поговорим о преимуществах.

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


@pytest.fixture(scope='function')
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()


def test_google_search_selenium(driver):
    driver.get('https://www.google.com')

    search_box = driver.find_element(By.NAME, "q")
    search_box.send_keys('selene github')
    search_box.send_keys(Keys.ENTER)

    expected_text = 'yashaka/selene: User-oriented Web UI browser tests in Python'
    h3_element = WebDriverWait(driver, 10).until(
        EC.visibility_of_element_located((By.TAG_NAME, 'h3'))
    )
    assert h3_element.text == expected_text
```

```python
from selene import browser, have


def test_google_search_selene():
    browser.open('https://www.google.com')

    browser.element('[name=q]').type('selene github').press_enter()

    browser.element('h3').should(have.text('yashaka/selene: User-oriented Web UI browser tests'))
```

Можем сразу заметить, что нам не пришлось настраивать браузер, а мы воспользовались готовым.

К тому же, проверка того, что элемент будет на странице заложена внутрь метода `should`

В целом меньше кода = меньше ошибок.

В этом примере мы видим, как легко и просто мы можем открывать веб-страницы, находить элементы и взаимодействовать с
ними, благодаря удобному синтаксису Selene.

А теперь подробнее поговорим о преимуществах библиотеки.

### Основные преимущества Selene

1. **Упрощенный синтаксис и автоматические ожидания**:
    - Selene предлагает более лаконичный и интуитивно понятный синтаксис по сравнению с оригинальным Selenium;
    - Библиотека автоматически управляет ожиданиями при работе с элементами, что снижает вероятность возникновения
      ошибок в тестах.

Рассмотрим тест, где нужно проверить тексты в таблице, а заодно мы проверим количество элементов в таблице по количеству
ожидаемых текстов:

Вариант для `selenium`

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait


@pytest.fixture
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()


def test_table_texts_selenium(driver):
    driver.get(
        'https://www.techlistic.com/2017/02/automate-demo-web-table-with-selenium.html')

    table = WebDriverWait(driver, 10).until(
        EC.visibility_of_element_located((By.ID, 'customers'))
    )

    tr_elements = table.find_elements(By.TAG_NAME, 'tr')

    expected_texts = ['Company Contact Country', 'Google Maria Anders Germany',
                      'Meta Francisco Chang Mexico',
                      'Microsoft Roland Mendel Austria',
                      'Island Trading Helen Bennett UK',
                      'Adobe Yoshi Tannamuri Canada',
                      'Amazon Giovanni Rovelli Italy', ]
    actual_texts = [element.text for element in tr_elements]
    assert actual_texts == expected_texts
```

С использованием `selene` это может выглядеть так:

```python
from selene import browser, have

def test_table_texts_selene():
    browser.open(
        'https://www.techlistic.com/2017/02/automate-demo-web-table-with-selenium.html'
    )
    
    browser.element('#customers').all('tr').should(
        have.exact_texts(
            'Company Contact Country',
            'Google Maria Anders Germany',
            'Meta Francisco Chang Mexico',
            'Microsoft Roland Mendel Austria',
            'Island Trading Helen Bennett UK',
            'Adobe Yoshi Tannamuri Canada',
            'Amazon Giovanni Rovelli Italy',
        )
    )
```

Пример, где ярко выражена необходимость указывать ожидание в `Selenium`

```python
import pytest
from selene import browser, have
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


@pytest.fixture(scope='function')
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()


def test_visible_text_without_wait_selenium(driver):
    driver.get('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    driver.find_element(By.ID, 'colorChange').click()

    visible_after = driver.find_element(By.ID, 'visibleAfter')

    assert "Visible After 5 Seconds" in visible_after.text


"""
Произойдет ошибка 'Visible After 5 Seconds' != ''
Если заменить проверку на assert visible_after.is_displayed()
 
 FAILED test_wait.py::test_visible_text_without_wait - assert False
+  where False = is_displayed()
+    where is_displayed = <selenium.webdriver.remote.webelement.WebElement (...)>.is_displayed
"""


def test_visible_after_text_with_wait(driver):
    driver.get('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    driver.find_element(By.ID, 'colorChange').click()

    visible_after = WebDriverWait(driver, 6).until(
        EC.visibility_of_element_located((By.ID, 'visibleAfter'))
    )
    assert "Visible After 5 Seconds" in visible_after.text


def test_visible_after_5_seconds_text_selene():
    browser.config.timeout = 10
    browser.open('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    browser.element('#colorChange').click()

    browser.element('#visibleAfter').should(
        have.text('Visible After 5 Seconds')
    )
```

В тесте, где используется `selene` пришлось обновить параметр `timeout` в конфигурации браузера, так как по умолчанию
значение стоит 4 секунды, а кнопка появляется через 5 секунд.

Но так как нам нужно дольше ожидать только один элемент `'#visibleAfter'`, можно тест переписать вот так:

```python
from selene import browser, have


def test_visible_after_5_seconds_text_selene_v2():
    browser.open('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    browser.element('#colorChange').click()

    browser.element('#visibleAfter').with_(timeout=browser.config.timeout * 2).should(
        have.text('Visible After 5 Seconds')
    )
```

Таким образом для всех других действий значение `timeout` останется 4 секунд, а для конкретного действия 8.

2. **Понятные и выразительные методы**
    - Методы Selene разработаны таким образом, чтобы они были самодокументируемыми и легко понятными.
      Например, `selene`'овский метод `type` (напечатать) против `send_keys` (отправить клавиши) на `selenium`
    - Еще различные удобства, такие как нажать клавишу `Enter` методом `press_enter()`, или навести мышь на элемент -
      `hover()` и прочее.

3. **Гибкость в выборе браузера**
    - Selene из 'коробки' поддерживает несколько браузеров, включая Chrome, Firefox и другие, что позволяет новичкам
      быстрее приступить к разработке тестов;
    - Достаточно будет в конфигурации браузера указать нужное имя драйвера, когда в `selenium` всё придется
      конфигурировать с нуля;
    - Для более тонкой настройки придётся передавать опции браузера, что будет аналогично тому, как мы бы настраивали
      драйвер в `selenium`;
    - По умолчанию в конфигурации браузера `selene` установлен `chrome`.

Открываем браузер Chrome и переходим на Google с помощью `selenium`:

```python
import pytest
from selenium import webdriver


@pytest.fixture(scope='function')
def driver():
    driver = webdriver.Chrome()
    yield driver
    driver.quit()


def test_google_search_selenium(driver):
    driver.get('https://google.com')
```

Аналогичный код на `selene`:

```python
from selene import browser


def test_google_search_selene():
    browser.open('https://google.com')
```

Как это работает в случае с `selene`?

На самом деле, `selene` делает примерно то же что и код на `selenium`, только более хитро, и главное "скрыто от нас",
чтобы сэкономить наше время.

Логика работы команды `browser.open()` примерно такая:

```python
if (browser.config.driver == None):
    if (browser.config.driver_name == 'chrome'):
        browser.config.driver = webdriver.Chrome()
    else (browser.config.driver_name == 'firefox')
    browser.config.driver = ...
...

browser.config.driver.get('https://google.com')
```

Соответственно, если бы мы хотели запустить тест в `firefox` а не `chrome`, нам бы пришлось написать уже чуть больше
кода, но все еще меньше чем на `selenium`:

```python
from selene import browser


def test_google_search_selene():
    browser.config.driver_name = 'firefox'
    browser.open('https://google.com')
```

В реальной жизни, конечно логика работы сложнее, ведь часто нам нужно больше чем только "тип браузера".
Например, нам может быть интересно запустить тест в браузере "в фоновом режиме", чтобы он не мельтешил перед глазами и
не отвлекал от других задач
(тесты могут бегать пол часа, и пока пройдут, мы можем выполнять другую работу).

Для этого используются опции драйвера:

```python
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options


@pytest.fixture(scope='session')
def chrome_headless():
    # Настраиваем опции для headless Chrome
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--window-size=1920,1080')

    driver = webdriver.Chrome(options=options)

    yield driver

    driver.quit()


def test_example(chrome_headless):
    chrome_headless.get("https://example.com")
    assert chrome_headless.title == "Example Domain" 
```

Сноска:
`Если не указать --window-size, браузер может запускаться с минимальными размерами, что иногда приводит к сбоям в тестах.`

Соответственно `selene` тоже должен это как то учитывать.

И так и есть, в логике упомянутых выше if-ов будет дополнительный шаг создания еще и `driver_options`, если вдруг мы не
передадим их явно, например, для того "headless-режима":

```python
from selene import browser, have
from selenium.webdriver.chrome.options import Options


def test_example():
    # Настраиваем опции для headless Chrome
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--window-size=1920,1080')
    browser.config.driver_options = options
    
    browser.open("https://example.com")
    
    browser.should(have.title('Example Domain'))
```

И при этом все еще мы получаем меньше кода, чем в случае с `seleneium`.
Более того, логика `selene` достаточно умна, чтобы, указывая именно "опции для firefox", `selene` догадался бы сам, что
именно этот браузер и надо открывать,
то есть достаточно следующего кода:

```python
from selene import browser
from selenium.webdriver.firefox.options import Options


def test_google_search_selenium():
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--width=1920')
    options.add_argument('--height=1080')

    browser.config.driver_options = options
    browser.open('https://google.com')
```

4. **Интуитивные проверки**

- Selene предлагает удобные методы для проверки состояния элементов, такие как `should(have.text(...))`, что упрощает
  написание тестов, которые требуют верификации результатов.

Так, например, `selene` предлагает как точное совпадение по тексту `should(have.exact_text())`, так и частичное
`have.text()`

Или можно удобно строить цепочки условий через операторы `or`, `and`, или передавать сразу список условий `by_and` или
`by_or`

```python
from selene import browser, have
from selene.core.condition import Condition


def test_with_multiple_conditions():
    browser.open('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    browser.element('#colorChange').click()

    browser.element('#visibleAfter').with_(
        timeout=browser.config.timeout * 2
    ).should(
        have.text('Visible').and_(have.text('After 5 Seconds'))
    )


def test_with_multiple_conditions_v2():
    browser.open('https://www.tutorialspoint.com/selenium/practice/dynamic-prop.php')

    browser.element('#colorChange').click()

    browser.element('#visibleAfter').with_(
        timeout=browser.config.timeout * 2
    ).should(
        Condition.by_and(have.text('Visible'), have.text('After 5 Seconds'))
    )
```

Другие вещи можно взять на самостоятельное изучение :)

Теперь, когда мы ознакомились с основными преимуществами, давайте перейдем к
следующему блоку, где мы рассмотрим конфигурацию браузера Selene.

## 3. Конфигурация браузера Selene

Давайте рассмотрим, как настроить конфигурацию браузера для автоматизации тестирования.

Selene предоставляет множество опций для настройки поведения браузера, что позволяет сделать тесты более гибкими и
адаптивными к различным условиям.

### Основные параметры конфигурации

1. **Выбор браузера**

Selene поддерживает несколько браузеров, таких как Chrome и Firefox. Чтобы указать, какой браузер использовать,
достаточно изменить одну строку в коде. Например, для использования Firefox:

```python
# список стандартных driver_name: chrome, firefox, edge, remote, appium
browser.config.browser_name = 'firefox'
# или можно так
browser.config.driver_name = 'edge'
```

2. **Настройки размера окна**

Вы можете настроить размер окна браузера перед запуском теста. Это полезно, чтобы проверить, как ваше приложение
выглядит на разных экранах. Например:

```python
browser.config.window_width = 1280
browser.config.window_height = 720
```

3. **Базовый URL**

Для упрощения тестирования вы можете задать базовый URL, который будет использоваться во всех тестах:

```python
browser.config.base_url = 'https://todomvc.com/examples/backbone'
```

Теперь вместо полного URL вы сможете использовать только относительный путь в своих тестах.

То есть, если не указывать базовый URL, тогда тест будет выглядеть следующим образом

```python
from selene import browser, have


def test_todomvc():
    browser.open('https://todomvc.com/examples/preact/dist/')

    browser.element('.new-todo').type('My first task').press_enter()

    browser.all('.todo-list li').should(have.exact_texts('My first task'))
```

Однако, если в конфигурации браузера настроить базовый URL, тогда можно будет написать так

```python
from selene import browser, have

browser.config.base_url = 'https://todomvc.com/examples/preact/dist'


def test_todomvc():
    browser.open('/')

    browser.element('.new-todo').type('My first task').press_enter()

    browser.all('.todo-list li').should(have.exact_texts('My first task'))
```

Еще нагляднее это можно сделать, если мы захотим проверить различные реализации `ToDoMVC`.

У каждой реализации свой относительный URL, тогда мы можем сохранить как базовый URL `https://todomvc.com`,
а относительные пути передавать как параметр для теста.

```python
import pytest
from selene import browser, have

browser.config.base_url = 'https://todomvc.com'


@pytest.mark.parametrize(
    'relative_url',
    ['/examples/preact/dist', '/examples/vue/dist/#', '/examples/angular/dist/browser/#/all'],
    ids=['preact', 'vue', 'angular']
)
def test_todomvc(relative_url):
    browser.open(relative_url)

    browser.element('.new-todo').type('My first task').press_enter()

    browser.all('.todo-list li').should(have.exact_texts('My first task'))
```

Это код параметризованного теста, где мы управляли значением относительной части URL.

Запустив этот тест можно заметить, что для реализаций на `angular` и `preact` тест будет выполнен успешно, а вот для
`vue` уже нет.

`Reason: AssertionError: actual visible_texts: ['My first task\nEdit Todo Input']`

Хотя внешне текста и не видно, но он всё равно есть в этом элементе.

4. **Параметры ожидания**

Selene позволяет настраивать время ожидания, чтобы ваш тест не завершился слишком быстро. Это поможет вам избежать
ошибок, если элемент на странице загружается дольше, чем ожидалось:

Глобально изменить настройку можно так:

```python
browser.config.timeout = 10  # время ожидания в секундах

```

Для конкретных элементов это можно настроить следующим образом:

```python
from selene import browser

browser.element().with_(timeout=10)  # или чуть хитрее, через относительное значение от browser.config.timeout * 1.5
```

Изменять для элементов полезно в том случае, когда весь UI грузится, например, до 4 секунд, а некоторые особо сложные
элементы
свыше 4 секунд. Вот на них и следует увеличивать значение `timeout`.

## 4. Поиск элементов и коллекций элементов

Для поиска элементов на странице вы можете использовать метод `element()` для поиска одного элемента и `all()` для
поиска нескольких:

```python
from selene import browser, have


def test_todos():
    browser.open('https://todomvc.com/examples/preact/dist')
    # Поиск одного элемента
    new_todo = browser.element('.new-todo')
    #
    new_todo.type('First task').press_enter()
    new_todo.type('Second task').press_enter()
    new_todo.type('Third task').press_enter()

    # Поиск всех элементов с определённым локатором
    todo_items = browser.all('.todo-list li')

    # Проверка коллекции элементов
    todo_items.should(
        have.exact_texts('First task', 'Second task', 'Third task'))
    # или, например, проверим, что в каждом элементе списка есть слово `task`
    todo_items.should(have.text('task').each)
```

Для работы с коллекцией элементов есть такие методы

### Получение одного элемента из коллекции

| **Метод**                             | **Описание**                                                                                            |
|---------------------------------------|---------------------------------------------------------------------------------------------------------|
| `[index]` или `element(index)`        | Возвращает элемент коллекции по индексу.                                                                |
| `element_by(condition)`               | Находит первый элемент, соответствующий условию, в коллекции.                                           |
| `element_by_its(selector, condition)` | Находит первый элемент коллекции, у которого есть вложенный элемент, соответствующий заданному условию. |

```python
from selene import browser, have


def test_todos():
    browser.open('https://todomvc.com/examples/preact/dist')

    new_todo = browser.element('.new-todo')
    new_todo.type('First task').press_enter()
    new_todo.type('Second task').press_enter()
    new_todo.type('Third task').press_enter()

    todo_items = browser.all('.todo-list li')

    first_element = todo_items[0]
    first_element_v2 = todo_items.element(0)
    first_element_v3 = todo_items.element_by_its('label', have.text('First task'))

    # проверим, что это тот же самый элемент
    assert first_element().id == first_element_v2().id == first_element_v3().id
```

### Работа со срезом коллекции

| **Метод**                                                          | **Описание**                                                                                                                                                                                              |
|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `[start:stop:step]` или `sliced(start: int, stop: int, step: int)` | Возвращает срез коллекции с возможностью указания начального и конечного индекса и шага.                                                                                                                  |
| `from_(start: int)`                                                | Возвращает коллекцию, начиная с элемента с указанным индексом.                                                                                                                                            |
| `to(stop: int)`                                                    | Возвращает коллекцию, заканчивая элементом с указанным индексом.                                                                                                                                          |
| `from_(start: int).to(stop: int)`                                  | Возвращает коллекцию, начиная с элемента с указанным индексом до `stop` шт. То есть, сначала вернется коллекция `[start:]`, а потом эта коллекция обрежется до `[:stop]` в два последовательных действия. |

```python
from selene import browser, have


def test_todos_collection():
    browser.open('https://todomvc.com/examples/preact/dist')
    new_todo = browser.element('.new-todo')
    new_todo.type('First task').press_enter()
    new_todo.type('Second task').press_enter()
    new_todo.type('Third task').press_enter()

    todo_items = browser.all('.todo-list li')

    todo_items[1:2].should(have.texts('Second task'))
    todo_items.sliced(start=1, stop=2, step=1).should(have.texts('Second task'))
    todo_items[1].should(have.texts('Second task'))

    todo_items[:1].should(have.texts('First task'))
    todo_items.to(1).should(have.texts('First task'))

    todo_items[1:].should(have.texts('Second task', 'Third task'))
    todo_items.from_(1).should(have.texts('Second task', 'Third task'))

    # Отдельно стоит выделить совместное использование from_().to()
    # [start:stop] != [start:][:stop]
    todo_items[1:][:2].should(have.texts('Second task', 'Third task'))
    todo_items.from_(1).to(2).should(have.texts('Second task', 'Third task'))
```    

## 5. Взаимодействие с элементами и коллекциями элементов

Можно выделить 2 основные группы взаимодействий:

- Выполнить действие
- Выполнить проверку условия

Причем вторая группа делится на три, но об этом чуть позже.

Поговорим о том, какие "Действия" можно выполнять

#### Основные методы класса `Element` в Selene

| **Метод**                                                 | **Описание**                                                                                                                    |
|-----------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `element(css_or_xpath_or_by)` или `s(css_or_xpath_or_by)` | Находит и возвращает первый элемент, соответствующий заданному CSS, XPath или селектору.                                        |
| `all(css_or_xpath_or_by)` или `ss(css_or_xpath_or_by)`    | Находит все элементы, соответствующие заданному CSS, XPath или селектору.                                                       |
| `set_value(value)`                                        | Устанавливает значение для элемента (например, текстовое поле).                                                                 |
| `type(text)`                                              | Вводит текст в элемент.                                                                                                         |
| `send_keys(*value)`                                       | Используется для более низкоуровневых операций, таких как загрузка файлов. Может имитировать ввод текста с клавиатуры.          |
| `press(*keys)`                                            | Имитирует нажатие клавиш на элементе.                                                                                           |
| `press_enter()`                                           | Имитирует нажатие клавиши Enter.                                                                                                |
| `press_escape()`                                          | Имитирует нажатие клавиши Escape.                                                                                               |
| `clear()`                                                 | Очищает содержимое элемента (например, текстового поля).                                                                        |
| `submit()`                                                | Отправляет форму, к которой принадлежит элемент.                                                                                |
| `click(*, xoffset=0, yoffset=0)`                          | Выполняет клик по элементу с возможностью указания смещения по осям X и Y.                                                      |
| `double_click()`                                          | Выполняет двойной клик по элементу.                                                                                             |
| `context_click()`                                         | Выполняет правый клик (context-click) по элементу.                                                                              |
| `hover()`                                                 | Наводит указатель мыши на элемент.                                                                                              |
| `execute_script(script_on_self, *arguments)`              | Выполняет JavaScript-скрипт на текущем элементе. Например, `element.execute_script('element.remove()')` - удалит элемент из DOM |
| `locate()` или просто `()`                                | Возвращает элемент WebElement для взаимодействия с ним через Selenium WebDriver.                                                |

`s()` и `ss()` является синонимами `element()` и `all()`, кому больше нравится стиль `selenide` из `Java` или `jQuery` в
`JavaScript`, где используются "доллары" для нахождения элементов.

#### Фильтрация коллекций в Selene

| **Метод**                       | **Описание**                                                                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------------------|
| `by(condition)`                 | Фильтрует коллекцию по условию для каждого элемента.                                                               |
| `by_their(selector, condition)` | Возвращает элементы коллекции, у которых есть вложенные/относительные элементы, соответствующие заданному условию. |
| `collected(finder)`             | Возвращает коллекцию, полученную через выполнение функции `finder` для каждого элемента коллекции.                 |
| `all(selector)`                 | Возвращает коллекцию всех элементов, найденных с помощью селектора внутри каждого элемента коллекции.              |
| `all_first(selector)`           | Возвращает коллекцию первых элементов, найденных с помощью селектора внутри каждого элемента коллекции.            |

#### Полезные атрибуты (@property) класса `Collection` в Selene

| **Атрибут** | **Описание**                                                                            |
|-------------|-----------------------------------------------------------------------------------------|
| `first`     | Возвращает первый элемент коллекции (эквивалент `[0]` или `element(0)`).                |
| `second`    | Возвращает второй элемент коллекции (эквивалент `[1]` или `element(1)`).                |
| `even`      | Возвращает все чётные элементы коллекции (индексы 1, 3, 5 и т.д.). Эквивалент `[1::2]`  |
| `odd`       | Возвращает все нечётные элементы коллекции (индексы 0, 2, 4 и т.д.). Эквивалент `[::2]` |

А теперь поговорим про проверку условиям

3. **Проверки с помощью `should`**

Метод `should()` позволяет проверить условие и при неуспешном результате бросить ошибку.

Именно таким образом в автотесте реализуется сравнение ожидаемого результата с актуальным.

Возвращаемся к примеру с поиском на странице Google

```python
from selene import browser, have


def test_google_search_selene():
    browser.open('https://www.google.com')

    browser.element('[name=q]').type('selene github').press_enter()

    browser.element('h3').should(have.text('yashaka/selene: User-oriented Web UI browser tests'))
```

В случае, если заменить ожидаемый текст на `Lorem ipsum dolor sit amet`, то тест завершится с ошибкой:

```text
E  Timed out after 4s, while waiting for:
E  browser.element(('css selector', 'h3')).has text Lorem ipsum dolor sit amet
E  
E  Reason: AssertionError: actual text: yashaka/selene: User-oriented Web UI browser tests in Python
```

Такое поведение будет для проверки состояния элемента и какого-либо условия.

Но еще есть два других метода: `matching()` и `wait_until()`, но там уже не вызывается исключение.

### 4. **Обработка условий при помощи методов `matching` и `wait_until`**

Оба метода возвращают булево значение: `True` или `False`, что позволяет обрабатывать условия.

Однако они имеют различия в поведении.

- **Метод `matching()`** выполняется "мгновенно", сразу проверяя условие и возвращая результат, без ожидания.
  Этот метод полезен, когда результат сравнения с условием ожидается без задержек. Например, можно использовать
  `matching()` для
  немедленной проверки отображения элемента на странице:

```python
if element.matching(have.text('Success')):
    print("Элемент содержит текст 'Success'.")
else:
    print("Элемент не содержит ожидаемый текст.")
```

- **Метод `wait_until()`** полезен, когда есть вероятность, что условие станет истинным через некоторое время.
  Этот метод ожидает успешного выполнения условия, возвращая False, если по истечении таймаута условие не было
  выполнено.
  `wait_until()` подходит для динамических страниц, где элементы загружаются с задержкой:

```python
if element.wait_until(have.text('Loaded')):
    print("Элемент загрузился с ожидаемым текстом.")
else:
    print("Элемент не загрузился с текстом в течение заданного времени.")
```

Когда мы изучили разницу между `should`, `matching` и `wait_until` можно поговорить о том, какие существуют условия для
проверки элементов, коллекций и браузера.

#### Условия для элементов

Подразумевается модуль **have**

Например, should(have.exact_text('sometext')))

| **Условие**                                    | **Описание**                                                                                             |
|------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| `exact_text(value)`                            | Проверяет, что текст элемента точно совпадает с заданным значением.                                      |
| `text(partial_value)`                          | Проверяет, что текст элемента содержит указанное частичное совпадение.                                   |
| `js_property(name: str).value(expected: str)`  | Проверяет, что у элемента есть указанный JavaScript-свойство с заданным значением.                       |
| `css_property(name: str).value(expected: str)` | Проверяет, что у элемента есть указанный CSS-свойство с заданным значением.                              |
| `attribute(name: str).value(expected: str)`    | Проверяет, что у элемента есть указанный атрибут с заданным значением.                                   |
| `value(text)`                                  | Проверяет, что значение элемента (например, текстовое поле) соответствует заданному значению.            |
| `value_containing(partial_text)`               | Проверяет, что значение элемента содержит указанное частичное совпадение текста.                         |
| `css_class(name)`                              | Проверяет, что у элемента есть указанный CSS-класс.                                                      |
| `tag(name: str)`                               | Проверяет, что элемент имеет указанный HTML-тег.                                                         |
| `tag_containing(name: str)`                    | Проверяет, что тег элемента содержит указанную подстроку.                                                |
| `no`                                           | Логическое отрицание. Например, should(have.no.text('123')) - проверит, что у элемент не содержит текста |

Для `js_property`, `css_property` и `attribute` не обязательно после писать проверку значения. Если её не написать, то
есть

вместо `have.attribute('name').value('some-name')` написать `have.attribute('name')`, тогда выполнится лишь проверка
существования такого атрибута, в данном случае `name`.

Если указать, будет выполнена `name` проверка, что значение атрибута `name` соответствует значению `some-name`

#### Состояние элементов

Подразумевается модуль **be**

Например, should(be.visible)

| **Условие** | **Описание**                                                                               |
|-------------|--------------------------------------------------------------------------------------------|
| `visible`   | Проверяет, что элемент виден пользователю.                                                 |
| `hidden`    | Проверяет, что элемент не виден пользователю.                                              |
| `selected`  | Проверяет, что элемент выбран. Обычно это касается чекбоксов и радиокнопок                 |
| `present`   | Проверяет, что элемент есть в DOM.                                                         |
| `in_dom`    | Проверяет, что элемент есть в DOM. (то же самое, что и `present`)                          |
| `existing`  | Проверяет, что элемент есть в DOM. (то же самое, что и `present`)                          |
| `absent`    | Проверяет, что элемента нет в DOM.                                                         |
| `enabled`   | Проверяет, что элемента есть атрибут `enabled` (включен)                                   |
| `disabled`  | Проверяет, что элемента есть атрибут `disabled` (выключен)                                 |
| `clickable` | Проверяет, что элемент виден пользователю и у элемента есть атрибут `enabled`              |
| `blank`     | Проверяет, что элемент должен быть пустым (value='' и text='')                             |
| `not_`      | Логическое отрицание. Например, should(be.not_.visible) то же самое, что should(be.hidden) |

#### Условия для коллекции элементов

| **Условие**                                                    | **Описание**                                                                                                                          |
|----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| `size(number: int)`                                            | Проверяет, что количество элементов в коллекции равно заданному числу.                                                                |
| `size_less_than(number: int)`                                  | Проверяет, что количество элементов в коллекции меньше заданного числа.                                                               |
| `size_less_than_or_equal(number: int)`                         | Проверяет, что количество элементов в коллекции меньше или равно заданному числу.                                                     |
| `size_greater_than(number: int)`                               | Проверяет, что количество элементов в коллекции больше заданного числа.                                                               |
| `size_at_least(number: int)`                                   | Проверяет, что количество элементов в коллекции не меньше заданного числа. (Возможно, будет заменен на `size_greater_than_or_equal`.) |
| `size_greater_than_or_equal(number: int)`                      | Проверяет, что количество элементов в коллекции больше или равно заданному числу.                                                     |
| `texts(*partial_values: Union[str, Iterable[str]])`            | Проверяет, что элементы коллекции содержат заданные тексты (частичные совпадения).                                                    |
| `exact_texts(*values: Union[str, Iterable[str]])`              | Проверяет, что элементы коллекции содержат заданные тексты (полное совпадение).                                                       |
| `values(*texts: Union[str, Iterable[str]])`                    | Проверяет, что элементы коллекции содержат заданные значения (точные совпадения).                                                     |
| `values_containing(*partial_texts: Union[str, Iterable[str]])` | Проверяет, что элементы коллекции содержат заданные значения, которые включают частичные совпадения.                                  |
| `no`                                                           | Логическое отрицание.                                                                                                                 |

#### Условия для браузера

| **Условие**                                          | **Описание**                                                                                                |
|------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| `url(exact_value: str)`                              | Проверяет, что текущий URL браузера точно совпадает с указанным значением.                                  |
| `url_containing(partial_value: str)`                 | Проверяет, что текущий URL браузера содержит указанный частичный текст.                                     |
| `title(exact_value: str)`                            | Проверяет, что заголовок страницы точно совпадает с указанным значением.                                    |
| `title_containing(partial_value: str)`               | Проверяет, что заголовок страницы содержит указанный частичный текст.                                       |
| `tabs_number(value: int)`                            | Проверяет, что количество вкладок в браузере соответствует заданному числу.                                 |
| `tabs_number_less_than(value: int)`                  | Проверяет, что количество вкладок в браузере меньше заданного числа.                                        |
| `tabs_number_less_than_or_equal(value: int)`         | Проверяет, что количество вкладок в браузере меньше или равно заданному числу.                              |
| `tabs_number_greater_than(value: int)`               | Проверяет, что количество вкладок в браузере больше заданного числа.                                        |
| `tabs_number_greater_than_or_equal(value: int)`      | Проверяет, что количество вкладок в браузере больше или равно заданному числу.                              |
| `js_returned_true(script_to_return_bool: str)`       | Проверяет, что скрипт в браузере вернул значение `True`. (Устарело, используйте `script_returned(...)`.)    |
| `js_returned(expected: Any, script: str, *args)`     | Проверяет, что скрипт в браузере вернул ожидаемое значение. (Устарело, используйте `script_returned(...)`.) |
| `script_returned(expected: Any, script: str, *args)` | Проверяет, что скрипт в браузере вернул ожидаемое значение.                                                 |
| `no`                                                 | Логическое отрицание.                                                                                       |

## 5. CSS Локаторы

В HTML и веб-автоматизации термины `tag`, `id`, `class name`, и `attribute` используются для описания различных
характеристик и способов идентификации элементов на веб-странице.

Разберем каждый из них:

1. **Tag (Тег)**
   Тег в HTML определяет тип элемента, например, `<div>`, `<p>`, `<h1>`, `<a>`, и т. д. Он определяет структуру
   веб-страницы и
   указывает, как элемент должен отображаться или использоваться.

Примеры тегов:

```html

<div>Это тег div</div>
<p>Это тег p (параграф)</p>
<h1>Это заголовок h1</h1>
<a href="#">Это ссылка (anchor tag)</a>
```

**Использование в поиске элемента:**

В `selene` можно искать элемент по его тегу, например:

```python
browser.element("h1")  # Поиск первого элемента с тегом <h1>
```

2. **ID (Идентификатор)**

Атрибут `id` является уникальным идентификатором элемента на странице.
На странице не должно быть двух элементов с одинаковым значением id. Этот атрибут полезен для точного и быстрого
поиска конкретного элемента.

На практике так бывает не всегда, но по крайней мере ID задуман быть уникальным.

Пример:

```html

<div id="main-container">Контент главного контейнера</div>
```

**Использование в поиске элемента:**

Для поиска по `id` можно использовать символ `#` перед значением, например:

```python
browser.element("#main-container")  # Поиск элемента с id="main-container"
```

3. **Class Name (Имя класса)**

Атрибут `class` (name) назначается элементу, чтобы применить к нему стили CSS или сгруппировать его с другими
элементами,
обладающими тем же классом.
В отличие от `id`, значение `class` (name) не обязано быть уникальным и может быть присвоено множеству элементов на
странице.

Имён классов может быть много, они разделены пробелом внутри атрибута `class`.

Важно именно то, что мы можем искать по одному из имён, а не по полному значению атрибута `class`.

Классы постоянно добавляются и убираются, поэтому не надо завязываться на весь атрибут целиком. Локатор станет хрупким.

Пример:

```html
<p class="intro-text other-class-name-1 another-name-3">Это первый параграф с именем классом intro-text. (в том
    числе)</p>
<p class="intro-text">Это второй параграф, в котором есть имя класса intro-text</p>
```

**Использование в поиске элемента:**

Для поиска по имени класса используется точка `.` перед значением класса:

```python

# поиск по атрибуту class, где селектор состоит из нескольких имен, каждое из которых передается слитно через точку
browser.element(".intro-text.other-class-name-1.another-name-3")  # -> Это первый параграф с именем классом intro-text. (в том числе)

# поиск по имени класса
browser.element(".intro-text")  # Поиск первого элемента с классом "intro-text"

# В данном случае вернется элемент параграф с текстом 'Это первый параграф с именем классом intro-text. (в том числе)', так как:
# 1. всегда возвращается первый найденный элемент
# 2. поиск был по всему DOM.

```

4. **Attribute (Атрибут)**

Атрибуты содержат дополнительную информацию об элементе и могут быть разными, например, `href`, `title`, `data-action`,
`aria-label`, `style`, и т. д.

Пример:

```html
<a href="https://example.com" title="Пример ссылки">Пример ссылки</a>
<button data-action="submit-form">Отправить</button>

```

**Использование в поиске элемента:**

Для поиска по атрибуту используется синтаксис `[attribute="value"]`:

```python
browser.element('[title="Пример ссылки"]')  # Поиск элемента с title="Пример ссылки"
browser.element('[data-action="submit-form"]')  # Поиск элемента с data-action="submit-form"
```

Если значение атрибута пишется без пробелов, то кавычки можно опустить

```python
browser.element('[data-action=submit-form]')  # Поиск элемента с data-action="submit-form"
```

5. **Локатор по атрибуту с частичным соответствием**

Рассмотрим на примере следующего HTML-дерева:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Example</title>
</head>
<body>
<form>
    <!-- Элемент, где name начинается с 'user' -->
    <input type="text" id="user_name" name="user_name" placeholder="Введите имя пользователя">

    <!-- Элемент, где name содержит 'user' -->
    <input type="tel" id="phone_contact" name="phone_user_contact" placeholder="Введите телефон">

    <!-- Элемент, где name заканчивается на 'user' -->
    <input type="email" id="contact_email" name="contact_email_user" placeholder="Введите email">
</form>
</body>
</html>

```

CSS позволяет выполнять частичный поиск по значению атрибута. Вот основные варианты:

- Используйте `^=` для поиска по атрибуту, начинающемуся с определенной строки

```python
input_user = browser.element('[name^="user"]')  # Найдет элемент с name="user_name"
```

- Используйте `$=` для поиска по атрибуту, заканчивающемуся на определенную строку

```python
input_email_user = browser.element('[name$="user"]')  # Найдет элемент с name="contact_email_user"
```

- Используйте `*=` для по атрибуту, содержащему подстроку

```python
input_users = browser.all('[name*="user"]')  # Найдет все элементы с подстрокой "user" в name
```

Этот запрос найдет и `user_name`, и `phone_user_contact`, и `contact_email_user`.

- Вы также можете использовать сложные селекторы, чтобы находить элементы по их порядку, например:

```python
second_button = browser.element('input:nth-of-type(2)')
```

Что вернет следующий элемент, так как он второй по списку. Отсчёт начинается с единицы, а не нуля.

```html
<input type="tel" id="phone_contact" name="phone_user_contact" placeholder="Введите телефон">
```

### Хрупкие локаторы

Хрупкие локаторы — это локаторы, которые могут легко сломаться из-за изменений в разметке страницы.

Это может произойти, если вы используете локаторы, зависящие от структуры документа или элементов, которые могут
изменяться (например, индексы или расположение элементов).

#### Лучшие практики для написания устойчивых локаторов:

1. Самым лучшим вариантом будет использовать тестовые атрибуты `[test-id]` и др., и если их нет, и нет возможности
   добавить (попросив разработчиков или самостоятельно), то только тогда переходить к `id`, `class`.

2. **Используйте уникальные атрибуты**: Старайтесь использовать идентификаторы (`id`), классы (`class`) и атрибуты,
   которые не изменяются и/или несут смысловую нагрузку.
    * Избегайте автоматически сгенерированных классов. Например, в Google поиске много подобных имён классов
      `<h3 class="bNg8Rb" ...>`

3. **Избегайте зависимостей от структуры**: Не полагайтесь на местоположение элемента в иерархии DOM. Используйте
   уникальные классы и атрибуты.

4. **Сокращайте количество компонентов**: Чем меньше элементов, от которых зависит локатор, тем лучше. Старайтесь
   использовать минимальное количество условий, сохраняя при этом читаемость селектора.

5. **Избегайте индексов**: Не используйте индексы (например, `:nth-child`), если это возможно, так как они могут
   изменяться при добавлении или удалении элементов.

6. **Проверяйте локаторы**: Всегда проверяйте ваши локаторы в консоли браузера, чтобы убедиться, что они работают и
   возвращают нужные элементы.
    - Например, в консоли браузера Chrome можно ввести команду `$('body')` для одного элемента и `$$('a')` для
      множества.

| **Плохой пример**               | **Хороший пример**              | **Почему?**                                                                                        |
|---------------------------------|---------------------------------|----------------------------------------------------------------------------------------------------|
| `div > div > span.product-name` | `[data-test-id="product-name"]` | Не полагайтесь на местоположение элемента в иерархии DOM. Используйте атрибуты вроде data-test-id. |
| `.bNg8Rb`                       | `[name="q"]`                    | Вместо сгенерированных имён классов надо использовать устойчивые атрибуты                          |
| `.items`                        | `#fruits .items`                | Если существует ID или говорящие атрибуты, то лучше иметь чёткую привязку контекста селектора      |

## 6. XPath

На примере простой веб страницы рассмотрим поиск элементов при помощи XPath

Пример HTML-кода и базовые примеры поиска элементов с использованием CSS-селекторов и XPath помогут увидеть разницу в
удобстве между этими методами:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sample Page</title>
</head>
<body>
<div id="header" class="main-header">
    <h1>Welcome to the Example Page</h1>
</div>
<div class="content">
    <p class="description" data-info="intro">This is a paragraph with some introductory text.</p>
    <p class="description" data-info="details">This paragraph provides additional details.</p>
    <a href="/about" class="link" data-action="about">About Us</a>
</div>
<div id="footer" class="main-footer">
    <p>&copy; 2024 Sample Company</p>
</div>
</body>
</html>
```

### **Примеры CSS-селекторов и XPath**

1. Поиск элемента по ID

* CSS: `#header`
* XPath: `//*[@id='header']`

2. Поиск элемента по имени тега и классу

* CSS: `div.content`
* *XPath: `//div[@class='content']`

3. Поиск элемента по атрибуту

* CSS: `[data-info="intro"]`
* XPath: `//*[@data-info='intro']`

4. Поиск элемента с частичным соответствием атрибута

* CSS: `[data-info^="intro"]` (начинается с)
* XPath: `//*[starts-with(@data-info, 'intro')]`

5. Поиск элемента по порядку

* CSS: `.description:nth-child(1)`
* XPath: `(//p[@class='description'])[1]`

6. Поиск по текстовому содержимому элемента

* CSS: Не поддерживается напрямую, необходимо использовать contains в селекторе JavaScript.
* XPath: `//p[contains(text(), 'introductory text')]`

### **Сравнение CSS и XPath**

| Критерий                 | CSS-селекторы                                                                             | XPath                                                                                                                                                                                              |
|--------------------------|-------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Простота записи**      | CSS-селекторы обычно короче и проще для чтения                                            | XPath может быть длиннее и требует больше скобок и операторов                                                                                                                                      |
| **Частичный поиск**      | CSS имеет специальные символы для частичного поиска (`^=`, `*=`, `$=`)                    | XPath использует функции `starts-with()`, `contains()`, `ends-with()` (в некоторых случаях сложнее)                                                                                                |
| **Поиск по тексту**      | Нет прямой поддержки                                                                      | `text()='Example Text'` для точного совпадения и `contains(text(), ...)` позволяет искать элементы по частичному содержанию текста                                                                 |
| **Поддержка браузерами** | Широко поддерживается браузерами и оптимизирован                                          | Может быть менее производительным в браузере, особенно при выполнении сложных запросов. Однако, учитывая общую медлительность современных веб-приложений, это вряд ли окажет значительное влияние. |
| **Навигация в DOM**      | Поддержка CSS ограничена выбором вложенных элементов и некоторыми родственными элементами | XPath позволяет двигаться вверх, вниз и по родственным элементам                                                                                                                                   |

Одним из значительных преимуществ XPath является возможность навигации вверх по дереву DOM, то есть поиск родительских
элементов.

Это особенно полезно, когда необходимо найти родителя или предка элемента, чтобы контекстно обработать его. В
CSS-селекторах такой возможности нет.

```html

<div class="product-container">
    <div class="product">
        <div>
            <label>20% off</label>
        </div>
        <div>
            <label>>$12</label>
        </div>
    </div>
</div>

<div class="product-container">
    <div>
        <div class="product">
            <span class="product-name">Product A</span>
        </div>
        <div>
            <label>buy now!</label>
            <label>$10</label>
        </div>
    </div>
</div>

```

В общем случае, когда мы явно хотим тот контейнер, в котором есть имя продукта, но нас интересует не только имя
продукта, но и другие его составляющие, например, цена.

И чтобы не строить разные локаторы под каждый атрибут товара, мы будем ориентироваться именно на `product-container`

```xpath
//*[@class='product-name']/ancestor::*[@class='product-container']
```

Здесь мы ищем элемент с классом `product-name` и поднимаемся вверх, чтобы найти его предка с классом
`product-container`.

В CSS-селекторах такой запрос не возможен напрямую. Для навигации в обратном направлении нужно использовать другие
стратегии, например, искать родительский элемент сначала и затем работать с его потомками.

Если мы хотим найти цену товара

```xpath
//*[@class='product-name']/ancestor::*[@class='product-container']//label[contains(text(), '$')]
```

В `selene`, чтобы найти родительский элемент, можно, например, написать так:

```python
from selene import browser

child_element = browser.element('#unique')

parent_element = child_element.element('..')
```

### Пример длинных локаторов CSS и XPath

Абстрактный пример, на практике, конечно, таким лучше не заниматься, так как это противоречит лучшим практикам
построения локаторов.

**CSS**

```css
div.container > section.main-content > div.product-list > div.product-item > span.product-name
```

**XPath**

```xpath
//div[contains(@class, 'container')]/section[contains(@class, 'main-content')]/div[contains(@class, 'product-list')]/div[contains(@class, 'product-item')]/span[contains(@class, 'product-name')]
```

В XPath часто приходится использовать более длинные локаторы из-за необходимости указывать атрибуты, в то время как
CSS-селекторы более лаконичны.

Однако, в случае сложных запросов или когда нужно обратиться к родительским элементам, XPath предлагает гибкость,
которой нет у CSS.

По факту, используя `CSS` и фильтрацию командами `selene`, нужда в `XPath` сводится к хождению по дереву вверх или
вправо/влево в единичных случаях в проекте.

В результате код становится более читаемым, ошибки — более понятными, а тесты будет легче поддерживать.

Автоматизация становится особенно полезной, когда фронтенд-разработчики запускают UI-тесты и анализируют причины их
падений.
Поскольку фронтенд-разработчики привыкли работать с CSS, использование CSS там, где это возможно, повысит шансы привлечь
их к использованию тестов и во многом добавит понимание того, на какой элемент указывает локатор в тесте.

## Ресурсы

### Selene

- [Официальная документация](https://yashaka.github.io/selene/)
- [Github](https://github.com/yashaka/selene)
- [Селениды: Быстрый Старт (Python)](https://autotest.how/selenides-quick-start-docs-md/)
- [Селениды в действии](https://autotest.how/selenides-in-action-docs-md/)

### pytest

- [Официальная документация](https://docs.pytest.org/en/stable/)

### CSS

- [Официальная документация Mozilla. Подробные описания селекторов с примерами.](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors)
- [Демонстрация различных селекторов](https://www.w3schools.com/cssref/trysel.php)
- [Игра-обучение](https://flukeout.github.io/#)
- [Расшифровка селекторов](https://kittygiraudel.github.io/selectors-explained/)

### Контакты

- [Чат QA.GURU](https://t.me/qa_guru_chat)
- [Чат QA - Automation](https://t.me/qa_automation)
- [Чат selene](https://t.me/selene_py_ru)
