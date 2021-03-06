#  Data Wrangling and Automating Summary Reports For Ultramarathon Finishing Times

  The marathon is a long distance running event, officially 42.195
  kilometers (or 26.2 miles) in length. However ultramarathons are
  longer, ranging from 50 kilometers (31.0686 miles) to 100 miles and
  beyond. This talk will tackle web scraping ultramarathon data with the
  R packages XML and rjson, transforming data with the plyr package,
  and automating summary reports with knitr and brew. We will explore
  parsing HTML using the XPath query language, transforming JSON objects
  to data frames, combining both brew syntax and knitr syntax together,
  and automating the creation of knitr reports.


## Ultramarathons
  * overview

    An ultramarathon is any running race longer than the traditional
    marathon length of 42.195 kilometers (26.219 mi). Typical race
    lengths are 50k, 50mi, 100k, 100 mi, and 24-hour events whereby
    the person who's run the farthest wins. Untramarathon's can also be
    differentiated by whether or not they're run on a track, pavement,
    on trails, or even over mountains, ala sky running.

    https://github.com/jeffreyhorner/RUNNING
    http://jeffreyhorner.blogspot.com/
    http://www.irunfar.com/

    So this talk will focus on a project I've recently undertaken that
    combines my love of running with software technology I've learned over
    the years and is relevant to those of you working with data: parsing
    data, or data wrangling if you want to be up on the latest buzz words,
    and programming in R, and the smallest bit of statistical analysis,
    mainly summary statistics and one linear prediction model (OLS).

    I'm a trained programmer with a bachelors in Computer Science, but
    I do not consider myself a statistician and I've got a lot to learn.

    So I'll focus more on the data aquisition, transformation,
    programming, and summary analysis, rather than the more formal
    seminar topics of statistical analysis and prediction and whathaveyou.

    I'll cover XML and it's cousin HTML, parsing HTML with the XPath
    query language, trasforming both XML and JSON, the javascript object
    notation, into R data. Using the plyr package and stitching this
    data into reports using markdown, brew, and knitr.

  * StumpJump blog post

    So I call myself a novice ultramarathoner. I've only raced three times
    at the 50k distance, and only in one annual race, The Rock/Creek
    StumpJump 50k held in Chatanooga, TN in October. And for those who
    are curious about just how much of a novice I am: I've done this
    race the past three years in a row, and my finishing times get slower
    and slower. I think I've figured out why: I don't run enough, so I'm
    fixing that in my training and I hope to do better this year. I'll
    keep you posted.

    So the outcomes of this project are blog posts, articles, essays, or
    whatever you want to call them, race reports that combine descriptions
    of the race with interesting data about the race, as much as I can
    get my hands on.

    http://ultrasignup.com/default.aspx

    So far, I've found a great resource called UltraSignup.com, a
    website that race directors can use to allow people to sign up for
    their races, but it also collects finishing times of each race,
    and historical finishing times. You can also search the site for
    particular people, i.e. runners, who you know or dont know, and see
    how old they are, male or female, where they say they're from, and all
    of their past races they've run and what their finishing times were.

    With only that data source, I've created a collection of plots used
    to summarize a race using finishing times, age, and gender data.

    * First up is the Number of Finishers for each year the race was
    contested.  A simple line plot of the total number of finishers in
    green, total number of men in blue, and total number of women in red.

    * Next is a two-parter, looking at the frequency of runners who
    finished within a 30 minute time period of each other. One for the
    current year the race was held (2013), and another for all the years
    the race was held.

    * Another look at finishing times but with a way to compare among
    each year for instance you can tell there was an increass in runners
    in 2010, mostly slow runners.

    * A look at the top 10 Finishing Times across years. Fastest year
    was 2011

    * A look at Age of Finishers each year.

    * A look at Finisher Frequency by Age.

    * Age Group Finishing Times - maybe a predictor for runners who are
    wanting to run a specific time.

  * Predicting UpChuck times given StumpJump time with OLS linear regression

    I won't say too much about this plot, but this is a simple linear
    prediction for the UpChuck 50k given a StumpJump 50k time. Basically
    I found all the runners who had finished both races, if they had run
    it multiple times, then I took the average. And that red line is
    what I call the Race Director's Recommendation. He basically said
    that you should add 2 hours to your StumpJump time to predict your
    UpChuck time. Of course the data say otherwise, right?

## XML, HTML, and XPath

  * XML overview - http://cran.r-project.org/web/packages/XML/index.html

    I'm sure many of you are familiar with the EXtensible Markup Language
    (XML) that defines a set of rules for encoding documents in a
    format that is both human-readible and machine-readible. and HTML is
    extremely similar to XML but was invented to make pretty web pages,
    and it turns out that the R XML package is very good at parsing both.

  * XPath overview - http://www.w3schools.com/xpath/

    XPath is a query language for selecting nodes from an XML (or HTML)
    document. It's very extensive, can even compute values from the
    content of the document. It's based on a tree representation of
    the document, and provides the ability to navigate around the tree,
    selecting nodes by a variety of criteria.

      [demo using books.xml]

    Let's start using a very simple xml document and highlighting
    the basics.

     x = xmlParse(file='books.xml')

    Now we have an object, not a file, but an XML object that we can
    now select nodes from.

    x

    the XML package has very nice features like the ability to print
    out objects which produce their textual representation.

    getNodeSet(x,'/')

    Our first XPath: this first part is called the 'location path',
    and it defines where to start navigating our xml tree. The '/'
    says start at the top of the tree. Also note that getNodeSet()
    returns an 'XMLNodeSet' list object, not the same as x, which is an
    'XMLInternalDocument'. No need to know exactly what that is, just that
    you'll want to know that what we'll now be computing on is a Node Set.

    getNodeSet(x,'/bookstore')

    Next in our sytax is a Node Test, expressions that define exactly
    what type of nodes we are interested in, for instance we want all
    root nodes that are named 'bookstore', and of course in this example
    there's only one.

    getNodeSet(x,'/bookstore/book')

    gets all 'book' nodes whose direct ancestor is 'bookstore' which is
    under the root node

    getNodeSet(x,'/bookstore/book/title[@lang="eng"]')

    Now we're using the last key piece of XPath called a Predicate. It's
    a way to further subset the nodes you want. In this case we want
    all title nodes with an attribute named lang with the value of 'eng'.

    getNodeSet(x,'/bookstore/book[last()]')

    Another predicate that says we want the last child node named 'book'
    in the bookstore

    getNodeSet(x,'//book')

    This path, the '//', will select any book node within the document,
    and this will be very important when used with predicates as a way
    to create shorter Xpaths rather than specifying the entire path of
    nodes up to the root of the document.

    lapply(getNodeSet(x,"//title"),xmlValue)

    You can get the string value of a node with xmlValue. Important
    that you've now extracted data from the XML object and no longer
    need XML related functions

    lapply(getNodeSet(x,"//title"),xmlAttrs)

    And here's how to get the attributes from each node.

    getNodeSet(x,"//title",fun=xmlAttrs)

    also takes a function that will operate on all nodes
    returned. Equivalent to the lapply statements above.

  * Parsing HTML

      Parsing HTML is very straightforward with the XML package, in fact
      all you have to do is call the htmlParse function rather than the
      xmlParse function. And since we are not parsing a file on our
      computer, we're going to use the RCurl package to download the
      contents of the url in real-time. It's a very easy software pattern
      copy and paste, especially  for the general case when the url is
      not password protected and expecting a human to be in control of
      the browser. But note that the RCurl package is very sophisticated
      and can handle the task of mimicing a human interaction by sending
      authentication credentials and managing stateful data like cookies
      and CGI data.

      library(RCurl) # http://cran.r-project.org/web/packages/RCurl/index.html
      ultraURL <- 'http://ultrasignup.com/entrants_event.aspx?did=26838'
      x <- htmlParse(getURL(ultraURL),asText=TRUE)

      getURL in the general case returns the web page as text, and we
      need to tell htmlParse that it's first argument is going to be
      text by setting asText=TRUE.

      Now you can go about using XPath just like before:

      getNodeSet(x,'//a[@class="event_selected_link"]')

      getNodeSet(x,'//a[@class="event_link"]')

      getNodeSet(x,'//table[@class="ultra_grid"]')
      

  * Using Chrome for HTML Inspection

      So how did I find out that the information I wanted from the webpage
      was the anchor nodes and the table node with the class ultra_grid?
      Easy, I used the chrome browser and it's built-in developer tools
      which allows you to inspect any part of a web page. All you do is
      right-click on a part of the page you're interested, and you're
      thrown into an HTML editor window that navigates to the exact part
      of the page in question.

      In the case of the links for the different race distances, I found
      that the class names were event_selected_link and event_link. Then
      I went to R and interactively explorted what kind of results I
      got. What I found when using this process is that the class and id
      elements are very key to subsetting the HTML and returning exactly
      what you want.

  * Caveat: Web pages change over time, so beware.

      HTML pages these days are meta-data rich, they're very expressive
      and you can really drill down into parts of the page you want by
      simply subsetting with an XPath that contains node tests on id or
      class attributes

      But note that web pages can change over time and it will cause
      your XPath's to fail, so be sure to test them often.

## rsjon
  * overview  - http://www.json.org/

    JSON (JavaScript Object Notation) is a lightweight data-interchange
    format. It is easy for humans to read and write. It is easy for
    machines to parse and generate. It is based on a subset of the
    JavaScript Programming Language, Standard ECMA-262 3rd Edition -
    December 1999. JSON is a text format that is completely language
    independent but uses conventions that are familiar to programmers of
    the C-family of languages, including C, C++, C#, Java, JavaScript,
    Perl, Python, and many others. These properties make JSON an ideal
    data-interchange language.

  * format

    JSON in it's simplest form consists of lists and vectors. And of
    course list elements can contain further lists and elements, so this
    maps really well into R.

    x = fromJSON(file='person.json')

    fromJSON also contains simple translation of JSON elements to native
    R classes

    x = fromJSON(file='product.json')

    class(x$properties$price$minimum)

    class(x$properties$price$required)

  * Downloading and parsing HTML with RCurl with fromJSONDF

    So we can download JSON payloads directly from the web as long as
    we know the url will return a JSON object. For instance I found
    that ultrasignup used such a thing for some of it's data on certain
    web pages:

    ultraURL = 'http://ultrasignup.com/service/events.svc/GetFeaturedEventsSearch/p=0/q=StumpJump' 
    x <- fromJSON(getURL(ultraURL))
    
  - use in queryEvent and fromJSONDF

    And once I found the structure of this particular JSON object,
    I could easily create an R function to turn the results of that
    search into a data frame. And with that I could easily create an
    interactive function to bypass the web browser altogether:

    queryEvent('StumpJump')

    x <- queryEvent('Rock Creek')

    queryEntrants(x[1,])

    queryRunner('Horner')
    
    runnerResults(queryRunner('Horner')[1,])

## plyr
  * overview - http://plyr.had.co.nz/

    plyr is a set of tools for a common set of problems: you need to
    split up a big data structure into homogeneous pieces, apply a
    function to each piece and then combine all the results back together.

    From his paper in JSS, Wickam states: (http://www.jstatsoft.org/v40/i01)

    In general, plyr provides a replacement for loops for a large set
    of practical problems, and abstracts away from the details of the
    underlying data structure. An alternative to loops is not required
    because loops are slow (in most cases the loop overhead is small
    compared to the time required to perform the operation), but because
    they do not clearly express intent, as important details are mixed in
    with unimportant book-keeping code. The tools of plyr aim to eliminate
    this extra code and illuminate the key components of the computation.

    And I tend to agree with him that code intentions are illuminated
    more clearly using plyr rather than R's apply family of functions.

    So, my intention of introducing plyr in this talk is not to go into
    great detail about all of it's functionality, but I want to illuminate
    how I used it with pretty nice results in this project.

  * ldply and ddply in queryEntrants

    Note that the first letter in a plyr function defines the input
    type to the function, and the second letter in the name defines it's
    output type.

    So scraping the Entrants page for data is one way I used plyr,
    specificaly ldply and ddply.

    First off, for any given race on ultrasignup, there might be multiple
    events contested. For instance at the StumpJump, they have a 50km and
    an 11 miler race. And when you first go to the Entrants page for the
    race, you cannot presume which distance is shown for the entrants
    listed on the page. But if you look hard enough at the green text
    you'll see that one is bolder than the other, so the distance that's
    bolded corresponds to the entrants on the page. And in order for us to
    programmatically scrape this page we need some sort of indication in
    the html to determine that. And we get it by inspecting with Chrome.

    Each one of those green distances turns out to be links that you can
    click on to get the entrants for that particular race distance, and
    we can harvest those into a data frame so that we can programmatically
    get all the entrants for all races.

    ultraURL <- 'http://ultrasignup.com/entrants_event.aspx?did=26838'
    x <- htmlParse(getURL(ultraURL,.opts=ultraOpts()),asText=TRUE)

    getNodeSet(x,'//a[@class="event_selected_link"]')

    getNodeSet(x,'//a[@class="event_link"]')

    ldply(
      getNodeSet(x,'//a[@class="event_selected_link"]|//a[@class="event_link"]'),
      function(i)
        data.frame(
          href=xmlAttrs(i)['href'],
          distance=xmlValue(i),
          selected=xmlAttrs(i)['class']=='event_selected_link',
          stringsAsFactors=FALSE)
    )

    So I've found that ldply and getNodeSet work very well together,
    obviously since getNodeSet returns a list, but also because the
    nodes that are being parsed are regularize, i.e each node has the
    same meta-data structure with the same attribute names and such but
    their values are different. And the results map well to data frames.

    e <- queryEvent('StumpJump')
    x <- queryEntrants(e)
   ddply(x,.(distance),.fun=summarise,numRunners=length(distance))
   ddply(x,.(distance,gender),.fun=summarise,count=length(gender))
   ddply(x,.(distance,state),.fun=summarise,count=length(state))
   x$age <- as.numeric(x$age)
   ddply(x,.(distance,gender),.fun=summarise,meanAge=mean(age,na.rm=TRUE))
   ddply(x,.(distance,gender),.fun=summarise,
     meanAge=mean(age,na.rm=TRUE),minAge=min(age),maxAge=max(age))

## brew
  * overview

    ‘brew’ provides a templating system for text reporting. The syntax
     is similar to PHP, Java Server Pages, Ruby's erb module, and
     Python's psp module.

    [show brew help file]

## markdown
  * overview - https://daringfireball.net/projects/markdown/

    ‘Markdown’ is a plain-text formatting syntax that can be converted
     to XHTML or other formats. Basically it renders markdown files to
     html files for you.

## knitr
  * overview - http://yihui.name/knitr/

    The knitr package was designed to be a transparent engine for dynamic
    report generation with R, solve some long-standing problems in Sweave,
    and combine features in other add-on packages into one package (knitr
    ≈ Sweave + cacheSweave + pgfSweave + weaver + animation::saveLatex
    + R2HTML::RweaveHTML + highlight::HighlightWeaveLatex + 0.2 * brew +
    0.1 * SweaveListingUtils + more).

  * customization in createReport

    So in my project I've created a function that will combine both knitr
    and markdown to ultimately create an HTML document from my knitr/markdown
    document, and here's the basic code to do such:

    fileFrag <- strsplit(reportFile,'\\.')[[1]]
    mdFile <- paste(fileFrag[1],'md',sep='.')
    htmlFile <- paste(fileFrag[1],'html',sep='.')
    knit(reportFile,envir=knitEnv)
    markdownToHTML(mdFile,htmlFile,options=mdOpts)

  * combining with brew and markdown in TeachingLab

    https://github.com/jeffreyhorner/TeachingLab

    And in another project I've combined knitr/markdown/brew
    together in a shiny app. The pattern here is to run your knitr
    document through brew first, and then knit, and then markdownToHTML:

     brewOutput <- character()
     brew(file,output=textConnection("brewOutput","w",local=TRUE),
        envir=parent.frame())
     htmlContent <- paste(try(knit2html(text=brewOutput)),forceMathJax,sep="\n")
    
## Elevation Plots
  - ggplot code

## Comments, Questions
