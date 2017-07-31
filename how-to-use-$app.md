$app is a global variable that you can call from anywhere in your code to access the main draw loop.

For example, from another class you could draw an ellipse from inside another class, you could do this:
class Person
  def show
    $app.ellipse(20,20,5,5)
  end
end


class MyApp < Propane::App
  def settings
    size(40,40)
  end
  def setup
    person = Person.new
    person.show
  end
end


