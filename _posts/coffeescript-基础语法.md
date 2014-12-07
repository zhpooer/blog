title: CoffeeScript 基础语法
date: 2014-11-20 14:13:25
tags:
---

# 基础语法 #

`> coffee –c –w status.coffee` Watch 

~~~~~~
<script src='http://coffeescript.org/extras/coffee-script.js'></script>
<script type='text/coffeescript' src='status.coffee'></script>
~~~~~~


~~~~~~coffeescript:
houseRoast = null

hasMilk = (style) ->
  switch style
    when "latte", "cappuccino"
      yes
    else
      no

makeCoffee = (requestedStyle) ->
  style = requestedStyle || 'Espresso'
  if houseRoast?
    "#{houseRoast} #{style}"
  else
    style

barista = (style) ->
  time = (new Date()).getHours()
  if hasMilk(style) and type > 12 then "NO!"
  else
    coffee = makeCoffee style
    "Enjoy your #{coffee}"
~~~~~~

~~~~~~coffeescript:
chuckNorris = 'Chuck Norris'
weak = 'weak'
chuckNorris is weak   # false
chuckNorris isnt weak # true

5 isnt 6 # true
5 is not 6 # false

# Unlike in JavaScript, the equality and inequality operators in CoffeeScript aren’t type
# coercive. They’re equivalent to === and !== in JavaScript:

'' == false # false
1 == '1' #true

4 + '3' # '43'
'apples' – 'oranges' # NaN
'3'*3 # 9

not (3%2) # false
not (4%2) # true
~~~~~~


## PROPERTY ACCESS #

~~~~~~
movie = title: 'Way of the Dragon', star: 'Bruce Lee'
myPropertyName = 'title'
movie[myPropertyName] # 'Way of the Dragon'
~~~~~~


## Types, existential, and combining operators #

~~~~~~
reference = null
reference == null # true
typeof null  # 'object'

pegasus? # false
roundSquare? # false
pegasus = 'Horse with wings'
pegasus? # true

typeof roundSquare
# 'undefined'
~~~~~~

## Statements as expressions #

~~~~~~

connectJackNumber = (number) ->
  "Connecting jack #{number}"
  
receiver = 'Betty'

switch receiver
  when 'Betty'
    connectJackNumber
    4
  when 'Sandra'
    connectJackNumber
    22
  when 'Toby'
    connectJackNumber
    9
  else
    'I am sorry, your call cannot be connected'


style = 'latte'
milk = switch style
  when "latte", "cappuccino"
    yes
  else
    no

## Try Catch
flyAway = (animal) ->
  if animal is 'pig'
    throw 'Pigs cannot fly'
  else
    'Fly away!'
    
peter = 'pig'
try
  flyAway peter
catch error
  error
finally
  'Clean up!'

# inline blocks
year = 1983
if year is 1983 then hair = 'perm'

lastDigit = 4
daySuffix = switch lastDigit
  when 1 then 'st'
  when 2 then 'nd'
  when 3 then 'rd'
  else 'th'

hair = 'permed' if year is 1983

~~~~~~

## Method #

~~~~~~
'haystack'.search 'needle' # -1
'haystack'.search 'hay' # 0
'haystack'.search 'stack' # 3


milkDrinks = 'latté,mocha,cappuccino,flat white,eiskaffee'
hakMilk = (style) ->
  milkDrinks.search(style) isnt -1

hasMilk 'mocha' # true
hasMilk 'espresso romano' # false

'haystack'.replace 'hay', 'needle' # 'needlestack'
milkDrinks.replace 'latté', 'latte'

'Cappuccino'.toLowerCase() # 'cappuccino'
'I am shouting!'.toUpperCase() # 'I AM SHOUTING!'


'Banana,Banana'.split /,/ # [ 'Banana', 'Banana' ]
'latte,mocha,cappuccino,flat white,eiskaffee'.split /,/ # [ 'latte', 'mocha', 'cappuccino', 'flat white', 'eiskaffee' ]

~~~~~~

## length, join, slice, and concat #

~~~~~~
fence = ['fence pail', 'fence pail']
fence.length # 2

fence[999] = 'fence pail'
fence.length # 1000

['double', 'barreled'].join '-' # 'double-barreled'
['good', 'bad', 'ugly'].slice 0, 2 # ['good', 'bad']

[0,1,2,3,4,5].slice 0,1 # [0]
[0,1,2,3,4,5].slice 3,5 # [3,4]

['mythril', 'energon'].concat ['nitron', 'durasteel', 'unobtanium'] # [ 'mythril', 'energon', 'nitron', 'durasteel', 'unobtanium' ]

~~~~~~

## In & Range & Comprehensions#

~~~~~~
'to be' in ['to be', 'not to be'] # true
living = 'the present'
living in ['the past', 'the present'] # true

[1..10] # [ 1,2,3,4,5,6,7,8,9,10 ]
[5..1]  # [ 5,4,3,2,1 ]

number for number in [9,0,2,1,0] # [9,0,2,1,0]
number + 1 for number in [9,0,2,1,0] # [10,1,3,2,1]
letter for letter in ['x','y','z'] # [x,y,z]

mix = (ingredient) -> "Mixing #{ingredient}"
for ingredient in ingredients when ingredient.search('flour') < 0
  mix ingredient

person for person in ['Kingpin', 'Galactus', 'Thanos', 'Doomsday'] by 2 # ['Kingpin', 'Thanos']
~~~~~~

# Functions #
~~~~~~
count = (text, delimiter) ->
  return 0 unless text
  words = text.split(delimiter || /,/)
  words.length


partyMessage = -> console.log "It's party time!"
setTimeout partyMessage, 1000

interval = setInterval partyMessage, 1000
clearInterval interval

multiple = (initial, numbers) ->
  total = initial or 1
  for number in numbers
    total = total * number
  total

sum = (numbers) ->
  total = 0
  for number in numbers
    total  = total + number
  total

## Accumulate
accumulate = (initial, numbers, accumulator) ->
  total = initial or 0
  for number in numbers
    total = accumulator total, number
  total

sum = (acc, current) -> acc + current
accumulate(0, [5, 5, 5], sum)

logArgument = (logMe='default') -> console.log logMe

~~~~~~

## do ##

~~~~~~
do ->
  name = 'Ren'
~~~~~~

# object #

~~~~~~

futurama =
  characters:[
    'Fry'
    'Leela'
    'Bender'
    'The Professor'
    'Scruffy'
  ]
quotes:[
  'Good news everyone'
  'Bite my shiny metal'
]

phonebook =
  numbers:
    hannibal: '555-5551'
    darth: '555-5552'
    hal900: 'disconnected'
    freddy: '555-5554'
    'T-800': '555-5555'
  list: ->
    "#{name}: #{number}" for name, number of @numbers
  add: (name, number) ->
    if not (name of @numbers)
      @numbers[name] = number
  get: (name) ->
    if name of @numbers
      "#{name}: #{@numbers[name]}"
    else
      "#{name} not found"

process.stdin.setEncoding 'utf8'
stdin = process.openStdin()
stdin.on 'data', (chunk) ->
  args = chunk.split(' ')
  command = args[0].trim()
  name = args[1].trim() if args[1]
  number = args[2].trim() if args[2]
  switch
    when 'add'
      res = phonebook.add(name, number) if name and number
    when 'get'
      console.log phonebook.get(name) if name

score for name, score of {bob: 152, john: 139, tracy: 209}

# 不选原型中
for own page, count of views
  sum = sum + count

views[key] ?= 0


# fat arrow => bind the value of this 
someElement.onClick = ->
  setTimeout =>
    this.innerHTML 'Got clicked', 1000

cassetteCopy = {}
for own property, value of cassette
   cassetteCopy[property] = value
~~~~~~

# class #

~~~~~~
class Views
  constructor: ->
    @pages = {}
  increment: (key) ->
    @pages[key] ?= 0
    @pages[key] = @pages[key] + 1
  total: () ->
    sum = 0
    for own url, count of @pages
      sum = sum + count
    sum

class Camera extneds Product
  megapixiels: ->
    @info.megapixiels || 'Unknow'

class Product
  instance = []  # 私有变量
  # 静态方法
  @find = (query) ->
    (product for product in instances when product.name is query)
  
  constructor: (name) ->
    instances = instances.concat [@]
    @name = name

class Product
  constructor: (name, cost) ->
    @name = name
    @cost = cost
  price: ->
    @cost

class Camera extends Product
  markup = 2
  price: ->
    super()*markup

Example::justAdded = -> "just added!"

## is equivalent to
Example.prototype.justAdded = -> "just added!"

Array::join = -> "Array::join was redefined"

include = (klass, module) ->
  for key, value of module
    klass::[key] = value
~~~~~~

# compose #

~~~~~~
compose = (f, g) -> (x) -> f g x
taxForProducts = compose tax, netProfit
loyaltyDiscountForUser = compose loyaltyDiscount, userSpend


# before
before = (decoration) ->
  (base) ->
    (params) ->
      decoration params
      base params

after = (decoration) ->
  (base) ->
    (params...) ->
      result = base params...
      decoration params...
      result

around = (decoration) ->
  (base) ->
    (params...) ->
     callback = -> base params...
     decoration ([callback].concat params)...

beforeAsync = (decoration) ->
  (base) ->
    (params..., callback) ->
      result = undefined
      applyBase = =>
        result = base.apply @, (params.concat callback)
      decoration.apply @, (params.concat applyBase)
      result

user?.contact?.phone?.home

~~~~~~

## Style ##

~~~~~~

highlight = (names...) ->
  for name in names
    color find(name), 'yellow'

teams = ['wolverines', 'sabertooths', 'mongooses']
highlight teams...

[a,b] = [b,a]

competitors = [
  name: 'wildcats'
  points: 5
  ,
  name: 'bobcats'
  points: 3
]

makeCompetition = ({max, sort}) ->
  {max, sort}

[first, middle..., last] = competitors

render = (user) ->
  """
  Home phone for #{user?.name?.first}: #{user?.contact?.phone?.home}
  """

contact = user?.contact?.phone?.home || 'Not provided'

class DuckRace
  constructor: (@ducks) ->
  go: ->
    duck.walk() for duck in @ducks

evens = numbers.filter (item) -> item%2 == 0

taxes = paid.map (item) -> item*0.1

# 链式调用
using = (object, fn) -> fn.apply object
using turtle, ->
  @forward 2
  @rotate 90
  @forward 4

x y z 4 # x(y(z(4)))

~~~~~~

# DSL #

~~~~~~

class Email
  SMTP_PORT = 25
  constructor: (option) ->
    ['from', 'to', 'subject', 'body'].forEach (key) =>
      @["_#{key}"] = option?[key]
      @[key] = (newValue) ->
        @["_#{key}"] = newValue
        @

scruffysEmail
  .to('agtron@coffeescriptinaction.com')
  .from('scruffy@coffeescriptinaction.com')
  .subject('Hi Agtron!')
  .body '''
    This is a test email.
  '''
  
scruffysEmail.send()
~~~~~~

## html ##

~~~~~~
loggedIn = -> true
doctype 5
html ->
  body ->
    ul class: 'info', ->
      li -> 'Logged in' if loggedIn()

doctype = (variant) ->
  switch variant
    when 5
      "<!DOCTYPE html>"

markup = (wrapper) ->
  (attributes..., descendents) ->
    attributesMarkup = if attributes.length is 1
      " " + ("#{name}='#{value}'" for name, value of attributes[0]).join ' '
    else
      ''
    "<#{wrapper}#{attributesMarkup}>#{descendents() || ''}</#{wrapper}>"

html = markup 'html'
body = markup 'body'
ul = markup 'ul'
li = markup 'li'
~~~~~~

## css ##

~~~~~~
emphasis = ->
  fontWeight: 'bold'

css
  'ul':
    emphasis()
  '.x':
    fontSize: '2em'

css = (raw) ->
  hyphenate = (property) ->
    dashThenUpperAsLower = (match, pre, upper) ->
      "#{pre}-#{upper.toLowerCase}"
    property.replace /([a-z])(A-Z)/g, dashThenUpperAsLower

    output = (
      for selector, rules of raw
        rules = (for ruleName, ruleValue of rules
          "#{hyphenate ruleName}: #{ruleValue};"
        ).join '\n'
        """
        #{selector} {
          #{rules}
        }
        """
    ).join '\n'
~~~~~~

~~~~~~
coffee = require 'coffee-script'

coffee.eval '2 + 4'
~~~~~~

# 兼容 #

~~~~~~
document.querySelector ?= (selector) ->
  if /^#/.test selector
    (document.getElementById (selector.replace /^#/gi, ''))
  else
    throw new Error 'Not supported by this implementation'

Object.create ?= (prototype, extensions) ->
  if extensions
    throw new Error 'Not supported by this implementation'
  else
    F = ->
  F.prototype = prototype
  new F()

~~~~~~

# Module #

~~~~~~

# -- controller.coffee –
class Controller
exports.Controller = Controller

# -- blog.coffee –-
controller = require './controller'
class Blog extends controller.Controller

exports.Post = Post
{Post} = require './post'
~~~~~~

# Cake #

~~~~~~
# Cakefile
{spawn} = require 'child_process'

## cake build

compile (directory) = ->
  coffee = spawn 'coffee', ['-c', '-o', "compiled/#{directory}", directory]
  
coffee.on 'exit', (code) ->
  console.log 'Build complete'

clean = (path, callback) ->
  exec "rm -rf #{path}", -> callback?()

forAllSpecsIn = (dir, fn) ->
  execFile 'find', [ dir ], (err, stdout, stderr) ->
    fileList = stdout.split '\n'
    for file in fileList
      fn file if /_spec.js$/.test file

runSpecs = (folder) ->
  forAllSpecsIn folder, (file) ->
    require "./#{file}"

task 'build', 'Compile the application', ->
  clean 'compiled', ->
    compile 'app', ->
      'Build complete'

task 'test' , 'Run the tests', ->
  clean 'compiled', ->
    compile 'app', ->
      compile 'spec', ->
        runSpecs 'compiled', ->
          console.log 'Tests complete'

## task 'deploy', ->
##   invoke 'build'
##   invoke 'test'

~~~~~~

~~~~~~
{describe, it} = require 'chromic'
{Post} = require '../../app/models/post'

describe 'Post', ->
  post = new Post 'A post', 'with contents'
  another = new Post 'Another post', 'with contents'
  it 'should return all posts', ->
    Post.all().length.shouldBe 2
  
  it 'should return a specific post', ->
    Post.get(post.slug).shouldBe 'a-post'
~~~~~~
