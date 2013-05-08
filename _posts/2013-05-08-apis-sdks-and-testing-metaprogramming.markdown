---
layout: post
title: APIs, SDKs and testing metaprogramming
category: posts
---

> I'm as restless as a willow in a windstorm
-- It Might As Well Be Spring

Gerunds can make for some ugly, confusing sentences if you are a lazy writer. Actually, reading up on it briefly, _metaprogramming_ might just be a straight up verbal noun. Damnit! I will leave it there and hopefully the shame will be improving. Shame: fighting moral turpitude for thousands of years!

At Givey we have a RESTful JSON API. We also have a [Rails SDK gem][1] for any client apps that want to use the API. Writing specs in the API for the JSON is awkward and full of compromises and will no doubt be the subject of a later post. Similarly, writing the SDK, that relies entirely on the same JSON, is not as straight forward as one might hope. Should a Rails model that represents an API JSON object define all its accessors on the fly, or should the accessors be hardcoded into the model? The first solution creates problems if your client app instantiates empty models e.g., `User.new` because without any data for the metaprogramming to operate on, there are no methods on the object and your `form_for`s are not happy. The second solution means that if you update your API objects everyone has to update the gem immediately. Not friendly.

What we settled on was a mix of the two: hardcode all the essential accessors that the objects require for useful new objects, and get the object to create accessors on the fly for any additional data it gets sent. All the models include the GiveyModel module:

{% highlight ruby linenos %}
module GiveyModel
  # stuff here
  
  def initialize(attributes = {})
    attributes.each do |name, value|
      self.class.send(:attr_accessor, name.to_sym) unless self.class.method_defined?(name.to_sym)
      # bunch of other stuff here
      send("#{name}=", value)
    end
  end
  
  # stuff here
end
{% endhighlight %}

Usually when you test an external service it's quite a job to keep up to date with any changes, but fortunately we own the external service so every time we update the JSON objects, we dump one of each that the gem uses and use it as a fixture in our specs. This feels like a nasty solution, not least because it relies on us remembering to copy the fixtures over, but so far we haven't come up with a better one. So, we have our fixture, which we use to create a [HashieMash][2] and then we can test that our models have the correct accessors, but the gem itself doesn't break if older versions are being used. Since our models use the same GiveyModel module, we wrote a shared example we can drop into the specs.

{% highlight ruby linenos %}
shared_examples "a givey_rails model with attr_accessors" do
  
  let(:subject)     { described_class.new }
  let(:object_hash) { send("#{described_class.name.demodulize.underscore}_hash") }

  it "should have all the correct accessors" do
    object_hash.keys.each do |method_from_json_key|
      should respond_to(method_from_json_key)
    end
  end
  
end

{% endhighlight %}

That seemed like a reasonable solution, until I noticed that _sometimes_ this shared example passed when it should have failed. _Sometimes passes_ is clearly the most revolting phrase in unit testing, and is most commonly evoked by changing spec running order. Since `attr_accessor` is a class method, I decided I needed to reload the class before each test, thusly:

{% highlight ruby linenos %}
shared_context "reload models" do

  before(:each) do
    GiveyRails.send(:remove_const, described_class.name.demodulize)
    load "#{described_class.name.underscore}.rb"
  end

end
{% endhighlight %}

And hooray! it all works. But it looks pretty horrible, right? It's doing a bunch of extra work just for the specs, so is it really testing the code that a production environment would see? I think it is, but it definitely stinks. I'd love to hear of a less smelly solution. Ideas, anyone?


[1]:  https://github.com/giveydev/givey-rails
[2]:  https://github.com/intridea/hashie
