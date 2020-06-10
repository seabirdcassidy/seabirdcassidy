---
layout: post
categories: writing
date: 2019-04-04T00:00:00.000Z
title: Making Custom StreamFields in Wagtail
description: Wagtail CMS tutorial on creating flexible StreamBlocks for your Django blog
---

Have you ever needed a specific content element in your Wagtail blog post builder? StreamFields provide us more options than the standard RichTextFields. StreamFields come out of the box with a handful of useful StreamBlocks, such as ListBlock, EmailBlock, TableBlock and ImageChooserBlock.

However, we have to build the more unique StreamBlocks ourselves. Here is a tutorial on how to make a custom image-within-a-link StreamField.

# Making StreamFields in Wagtail

## Requirements

I am using Wagtail version 2.1.3 and Django 1.11.

## Example: Image With A Hyperlink

You want to create a StreamBlock for an image that has a hyperlink on it. For example, the HTML equivalent would look like:

```html

<a href="https://www.google.com"><img src="/assets/images/banner.jpg" /></a>
```

When users click the image they go to an external site. By default, this functionality doesn't exist within Wagtail. We have to build it. So you have a simple `models.py` that looks like this.

```python

from django.db import models

from wagtail.core.models import Page
from wagtail.core.fields import StreamField
from wagtail.core import blocks
from wagtail.admin.edit_handlers import FieldPanel, StreamFieldPanel
from wagtail.images.blocks import ImageChooserBlock

class BlogPage(Page):
    author = models.CharField(max_length=255)
    date = models.DateField("Post date")
    body = StreamField([
        ('heading', blocks.CharBlock(classname="full title")),
        ('paragraph', blocks.RichTextBlock()),
        ('image', ImageChooserBlock()),
    ])

    content_panels = Page.content_panels + [
        FieldPanel('author'),
        FieldPanel('date'),
        StreamFieldPanel('body'),
    ]
```

We will want to edit the `models.py` file to include an import from our app's core.blocks (we will create that file next) and the new block for the body. It will now look like this:

```python

from django.db import models

from wagtail.core.models import Page
from wagtail.core.fields import StreamField
from wagtail.core import blocks
from wagtail.admin.edit_handlers import FieldPanel, StreamFieldPanel
from wagtail.images.blocks import ImageChooserBlock
from my_blog_project.core.blocks import ImageWithLink

class BlogPage(Page):
    author = models.CharField(max_length=255)
    date = models.DateField("Post date")
    body = StreamField([
        ('heading', blocks.CharBlock(classname="full title")),
        ('paragraph', blocks.RichTextBlock()),
        ('image', ImageChooserBlock()),
        ('image_with_link', ImageWithLink()),
    ])

    content_panels = Page.content_panels + [
        FieldPanel('author'),
        FieldPanel('date'),
        StreamFieldPanel('body')
    ]
```

Now we'll head over to our `blocks.py` file and add our new ImageWithLink class. Make sure you are importing the ImageChooserBlock also at the top, as we are building off of it.

```python

from django.db import models
from wagtail.core import blocks
from wagtail.images.blocks import ImageChooserBlock


class ImageWithLink(blocks.StructBlock):
    """ Image with a clickable link on it """
    image = ImageChooserBlock(label="Image", required=True, help_text="Must be at least 2048 x 1535 for full screen images"))
    link = blocks.URLBlock(label="Link", required=True, help_text="Include https:// or http://")

    class Meta:
        template = 'core/blocks/imagewithlink.html'
        form_classname = 'imagewithlink'
        icon = 'picture'
```

And, following the path to completion, we need to create a file called `core/templates/core/blocks/imagewithlink.html`. This will be the template for that content element. It could look something like this, this is where you get creative:

```html

<a href="{% raw %}{{ self.link }}{% endraw %}" target="_blank">
  {% raw %}
  {% image self.image %}
  {% endraw %}
</a>
```

Now the last thing to do, considering we updated the models, is to run `./manage.py makemigrations` and `.manage.py migrate` in the terminal directory of your Django project. Next time you log into Wagtail admin to create a blog post you'll see that the StreamField panel has a new StreamBlock inside it - an image with a link.
