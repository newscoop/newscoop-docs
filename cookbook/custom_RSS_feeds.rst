Custom RSS feeds
----------------

*Introduced as from Newscoop Version: 4.4.7*

Newscoop provides two urls which can be used as RSS feed endpoints:

 * ``/{languageCode}/feed/`` example: /en/feed
 * ``/{languageCode}/feed/{feedName}`` example: /en/feed/sport

``/{languageCode}/feed/`` is a shortcut for ``/{languageCode}/feed/default``.

Newscoop will look for current publication theme ``_feed/`` directory and will try to load ``{feedName}.tpl`` template.

Template will have (under ``$gimme->language``) attached language matching {languageCode}. For example: /en/feed will load ``_feed/default.tpl`` and in template file Language will be English.

In system_templates you can find example for default feed template (it will be used as a fallback when Your theme will not provide own one):

We assume that your Article Type have filed called ``deck`` (with lead for article).

.. code-block:: xml

    <rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom" xmlns:media="http://search.yahoo.com/mrss/">
      <channel>
        <title>{{$gimme->publication->meta_title}}</title>
        <link>https://{{$gimme->publication->site}}</link>
        <description>{{$gimme->publication->meta_description}}</description>
        <language>{{ $gimme->language->code }}</language>
        <copyright>Copyright {{$smarty.now|date_format:"%Y"}}, {{$gimme->publication->name}}</copyright>
        <lastBuildDate>{{$smarty.now|date_format:"%a, %d %b %Y %H:%M:%S"}} +0100</lastBuildDate>
        <ttl>60</ttl>
        <generator>Newscoop</generator>
        <image>
          <url>https:{{ url static_file="_img/logo.png" }}</url>
          <title>{{$gimme->publication->meta_title}}</title>
          <link>http://{{$gimme->publication->site}}</link>
        </image>
        <atom:link href="http://{{ $gimme->publication->site }}{{ generate_url route="newscoop_feed" }}" rel="self" type="application/rss+xml" />
        {{ list_articles length="30" name="recent_articles" ignore_section="true" ignore_issue="true" ignore_publication="true" order="bypublishdate desc"}}
        <item>
          <title>{{$gimme->article->name|html_entity_decode|regex_replace:'/&(.*?)quo;/':'&quot;'}}</title>
          <link>https://{{ $gimme->publication->site }}/+{{ $gimme->article->webcode }}</link>
          <description>
            {{ list_article_images length="1" }}
              &lt;img src="//{{$gimme->publication->site}}/{{$gimme->article->image->getImageUrl(600, 400)}}" border="0" align="left" hspace="5" /&gt;
            {{ /list_article_images }}
            {{$gimme->article->deck|strip_tags:false|strip|escape:'html':'utf-8'}}
            &lt;br clear="all"&gt;
          </description>
          <category domain="{{ uri name="section" }}">{{$gimme->section->name}}</category>
          <atom:author>
            <atom:name>{{ $gimme->article->author->name }}</atom:name>
          </atom:author>
          <pubDate>{{$gimme->article->publish_date|date_format:"%a, %d %b %Y %H:%M:%S"}} +0100</pubDate>
          <guid isPermaLink="true">{{ uri name="article" }}</guid>
        </item>
        {{/list_articles}}
      </channel>
    </rss>

*RSS feed response content type:*

Response for all rss feeds from rss controller will have ``application/rss+xml; charset=UTF-8`` Content-Type header.

*Generate url's to feeds from template:*

.. code-block:: smarty

    // Default language is English
    {{ generate_url route="newscoop_feed" }} => /en/feed/


.. code-block:: smarty

    // Custom language, default feed template
    {{ generate_url route="newscoop_feed" parameters=['languageCode'=> 'pl' ] }} => /pl/feed/

.. code-block:: smarty

    // Custom language and custom feed template
    {{ generate_url route="newscoop_feed" parameters=['languageCode'=> 'pl', 'feedName'=> 'template' ] }} => /pl/feed/template/
