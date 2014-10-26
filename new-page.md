---
layout: page
title: "New Page"
description: ""
---
{% include JB/setup %}

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <P>{{ post.excerpt }}</p>
    </li>
  {% endfor %}
</ul>
<h3>高亮代码</h3>
{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}

{% highlight python %}
""" This is a test method """
def test
    va = "htest"




{% endhighlight %}


{% highlight ruby %}
# This is a test
# Test
{% endhighlight %}
