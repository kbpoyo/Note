  

a = "hello world"

  

b = "hhhhh"

  

c = [[hel jald 

asdasd 

asdas

debug.getuservalue

asd]]

  

fun = function(a, b, c)

    print(a, b, c)

    return a..b

end

  

print("\n", fun(b, a, c))

  

d = {1, "ac", {}, fun}

  

print(d[4]("hello", "hhhhh"))

  

print(#d)

  

table.insert(d, "fu")

  

print(#d)

  

table.insert(d, 1, "helo")

  

print(d[1])

  

print("remove：", table.remove(d, 1))

  

print(d[1])

  
  

t = {

    a = 1,

    b = "1234",

    c = function()

    end,

  

    hello = 4567,

  

    ["world"] = 789

}

  

print(t["hello"])

  

print(t.hello)

  

print(t.world)

  

print(_G["a"])

  
  

a = true

b = false

  

if b then

    print(a and b)

    print(a or b)

    print(not a)

elseif 2<1 then

    print(2>1)-- body

elseif 0 then

    print(0 == true)

end

  
  

print(1~=2)

  
  

for i = 1, 10 do

    print(i)

end

  

for i = 1, 10, 2 do

    print(i)

end

  

for i = 1, 10 do 

    i = i + 1

    print(i)

end

  

local x = 1

  

while x < 10 do

    x = x + 1

    print(x)

end

  

s = string.char(0x30, 0x31)

  

n = string.byte(s, 2)

  

print(s, n, #s)