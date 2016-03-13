---
title: "Capybara, save_and_open_page for the lazy!"
layout: post
date: 2016-03-05 14:00
tag:
- testing
- rails
blog: true
---


When I'm writting Rails integration tests with capybara, every once
in a while there is something not working as I wish.
When misfourtune occures, usually I want to see how my
rendered page looks like **save_and_open_page** to the rescue!

Lets for the moment imagine that we are testing site navigation links.
When Guest clicks on **About** navigation link, he/she should see **About
page**.

{% highlight ruby %}
describe "Guest sees the about page" do
  it "when about page link is clicked" do
    visit root_path

    click_link "About"

    expect(page).to have_css ".title", text: "About page"
  end
end
{% endhighlight %}


After running test, it is **failling**

Hmm....that is strange!
Lets see what my page looks like, just add save_and_open_page method
call!

{% highlight ruby %}
describe "Guest sees the about page" do
  it "when about page link is clicked" do
    visit root_path

    click_link "About"

    save_and_open_page
    expect(page).to have_css ".title", text: "About page"
  end
end
{% endhighlight %}

Capybara will open browser and render my current page.
**Sueprise....surprise**, there is no about link rendered.

**Thanks save_and_open_page you just save me a lot of the debugging!**

Only thing that bugs me with save_and_open_page is size of method name,
I'm just lazy typing it.
It would be a lot nicer if I could just type **page!**.

Well... I could, lest write small Capybara extenstion and add it to
spec/support directory.

{% gist dixpac/20b9e4f90529268b15aa %}

Now I could just type
**page!**

{% highlight ruby %}
describe "Guest sees the about page" do
  it "when about page link is clicked" do
    visit root_path

    click_link "About"

    page!
    expect(page).to have_css ".title", text: "About page"
  end
end
{% endhighlight %}
