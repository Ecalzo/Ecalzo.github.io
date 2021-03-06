---
layout: post
title: Templated SQL with Jinja
---

In data warehousing, we often encounter repetitive processes that can benefit from templating. This is a simple example of creating a `COPY INTO` statement using some JSON.

{% highlight jinja %}
{% raw %}
{# jinja_template.j2 #}
COPY INTO {{ table.target_name }} 
SELECT 
{% for col in table.columns %}
    {{ col.name }} as {{ col.alias }} {% if not loop.last %},{% endif %}
{% endfor %}
FROM {{ table.s3_stage }}
{% endraw %}
{% endhighlight %}

{% highlight json %}
// config.json
{
    "table": {
        "target_name": "transactions.app_payments",
        "columns": [
            {
                "name": "date",
                "alias": "payment_date"
            },
            {
                "name": "price",
                "alias": "app_price"
            },
            {
                "name": "ts",
                "alias": "purchase_ts"
            }
        ],
        "s3_stage": "s3://import_bucket/purchase_date/"
    }      
}
{% endhighlight %}

And now for our python driver program

{% highlight python %}
from jinja2 import Template
import json

json_config = json.loads(open("config.json").read())
template = Template(open("jinja_template.j2").read())
rendered_sql = template.render(json_config)
print(rendered_sql)
{% endhighlight %}


Here's a repl to play with:

{% include repls/jinja_repl.html %}
