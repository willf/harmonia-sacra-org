require 'rubygems'
require 'erb'
require 'csv'
require 'rexml/document'
require 'rake'

SE = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'.split("")

TEMPLATE_DIR = File.join(File.dirname(__FILE__),'templates')
SITE_DIR = File.dirname(__FILE__)
DATA_DIR = File.join(File.dirname(__FILE__), 'data')
MIDI_DIR = File.join(SITE_DIR, 'midis')
MP3_DIR = File.join(SITE_DIR, 'mp3s')
HS_4_DIR = File.join(SITE_DIR, 'hs_4_pdf')
HS_7_DIR = File.join(SITE_DIR, 'hs_7_pdf')
DIR_IMG = File.join(SITE_DIR, 'hs_tune_images')

def tf(name)
  File.join(TEMPLATE_DIR, name)
end

def sf(name)
  File.join(SITE_DIR, name)
end

def df(name)
  File.join(DATA_DIR, name)
end

def read_tunes_from_tabbed
  File.open(df('handbook.txt')).readlines.map do |line|
    line.chomp.split(/\t/)
  end.reject{ |l| l.size <=1  }
end

def read_tunes_from_xml
  doc = REXML::Document.new(File.new(df('handbook.xml')))
  doc.elements.each('*/tune'){}.map do |tune_el|
    $tunes = {}
    tune_el.elements.each do |element| 
      txt = element.text
      txt ? (txt=txt.strip) : (txt = '')
      $tunes[element.name.strip.downcase]=txt if element.name
    end
    $tunes
  end
end 

alias read_tunes read_tunes_from_xml

$tunes = read_tunes


# def to_xml
#   page = HSPage.new
#   page.tunes = page.read_tunes_from_tabbed
#   puts ERB.new(File.open(tf('xml.erb')).readlines.join,0,"%<>").result(binding)
# end

class HSPage 
#  Page = 0
#  Tune = 1
#  Metre = 2
#  Doremi = 3
#  Firstline = 4
#  AltName = 5
#  Composer = 6 
#  ComposerDate = 7
#  ComposerNotes = 8
#  Author = 9
#  AuthorDate  = 10
#  AuthorNotes = 11

  Page = 'page'
  Tune = 'tune'
  Metre = 'metre'
  Doremi = 'doremi'
  Firstline = 'firstline'
  AltName = 'altname'
  Composer = 'composer'
  ComposerDate = 'composerdate'
  ComposerNotes = 'composernotes'
  Author = 'author'
  AuthorDate  = 'authordate'
  AuthorNotes = 'authornotes'
  
  attr_accessor :title, :subtitle, :filename
  
  def initialize
    @title = 'Harmonia Sacra'
    @subtitle = 'A seven-shape tunebook in the Mennonite tradition'
    @filename = 'index.html' 
  end
  
  def gen
    puts sf(@filename)
    File.open(sf(@filename), 'w'){|out|
      @out = out
      gen_head 
      @out.puts "<body>
        <div id=\"container\">"
      
      gen_header
      gen_left
      @out.puts "<div id='main'>\n"
      gen_body
      @out.puts "</div>\n"      
      gen_footer
      @out.puts "</div></body>\n</html>"
    }
  end
  def open_template(name)
    ERB.new(File.open(tf(name)).readlines.join,0,"%<>")
  end
  def gen_template(name)
    @out.puts(open_template(name).result(binding))
  end
  def gen_head
    gen_template('head.erb')
  end
  def gen_header
    gen_template('header.erb')
  end  
  def gen_left
    gen_template('left.erb')
  end                 
  def gen_body
    gen_template('basic_body.erb')
  end
  def gen_footer
    gen_template('footer.erb')
  end 
  
  def split_page(page1)
     _, n1, p1 = page1.split(/([[:digit:]]+)/) 
     [n1.to_i, p1]
   end 
  def finess_page(pagelink)
    return pagelink if File.exists?(sf(pagelink))
    newlink = File.join(File.dirname(pagelink)[0..-1], ((File.basename(pagelink,'.*')[0..-1]+'b')+ File.extname(pagelink)))
    return newlink if File.exist?(sf(newlink))
    puts "warn: page #{pagelink} at #{sf(pagelink)} not found; neither was #{newlink} at #{sf(newlink)}; using #{pagelink}. "
    pagelink
  end
end 

class HomePage < HSPage
  def initialize 
    @title = 'Harmonia Sacra'
    @subtitle = 'A seven-shape tunebook in the Mennonite tradition'
    @filename = 'index.html' 
  end 
  
  def gen_body
    gen_template('welcome_body.erb')
  end
end 

class CopyrightPage < HSPage
  def initialize 
    @title = 'Harmonia Sacra'
    @subtitle = 'Copyright details'
    @filename = 'copyright.html' 
  end 
  
  def gen_body
    gen_template('copyright_body.erb')
  end
end

class TitleIndexPage < HSPage
  def initialize 
    @title = 'Tune Index'
    @subtitle = 'Tunes alphabetically indexed'
    @filename = 'title_index.html' 
  end 

  def gen_body
    tunes = $tunes.sort{ |a,b| a[Tune].to_s <=> b[Tune].to_s } 
    @out.puts "<h2>Tunes</h2>"
    @out.puts "<ul>"
    tunes.each do |t| 
      n,p = split_page(t[Page])
      @out.puts "<li><a href=\"#{t[Page]}.html\">#{t[Tune]}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" if t[Tune] and t[Tune].size > 0
    end
    @out.puts "</ul>"
  end
end     

class AltTitleIndexPage < HSPage
  def initialize 
    @title = 'Alternative Tune Index'
    @subtitle = 'Tunes alphabetically indexed'
    @filename = 'alt_title_index.html' 
  end 

  def gen_body
    tunes = $tunes
    alt_tunes = []
    tunes.each do |tune|
     alt_tunes << [:std, tune[Page], tune[Tune], tune[Firstline]]
     tune[AltName].strip.split(';').each{|n| alt_tunes << [:alt, tune[Page], n, tune[Firstline], tune[Tune]]}
    end
    alt_tunes = alt_tunes.sort {|a,b| a[2] <=> b[2]} 
    @out.puts "<h2>Tunes and Alternative Tune Names</h2>"
    @out.puts "<p>Tune names as they appear in the <i>Harmonia Sacra</i> are in a standard face; Alternative names are in <i>italic face</i>.</p>"
    @out.puts "<ul>"
    alt_tunes.each do |alt| 
      n,p = split_page(alt[1])
      if alt[0]==:std
        @out.puts "<li><a href=\"#{alt[1]}.html\">#{alt[2]}</a> <a href=\"#{alt[1]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{alt[3][0..50]}...)</li>" if alt[2] and alt[2].size > 0
      else
        @out.puts "<li><a href=\"#{alt[1]}.html\"><i>#{alt[2]}</i></a> (<a href=\"#{alt[1]}.html\">#{alt[4]}</a>) <a href=\"#{alt[1]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{alt[3][0..50]}...)</li>" if alt[2] and alt[2].size > 0
      end
    end
    @out.puts "</ul>"
  end
end 

class FirstIndexPage < HSPage
  def initialize 
    @title = 'First Line Index'
    @subtitle = 'Tune alphabetically indexed by first line'
    @filename = 'first_line_index.html' 
  end 

  def gen_body
    tunes = $tunes.sort{ |a,b| a[Firstline].to_s <=> b[Firstline].to_s } 
    @out.puts "<h2>First line</h2>"
    @out.puts "<ul>"
    tunes.each do |t| 
      n,p = split_page(t[Page])
      @out.puts "<li><a href=\"#{t[Page]}.html\">#{t[Firstline]}</a> (<a href=\"#{t[Page]}.html\">#{t[Tune]}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>)</li>" if t[Tune] and t[Tune].size > 0
    end
    @out.puts "</ul>"
  end
end 

class MinutesPage < HSPage
  def initialize 
    @title = 'Minutes of Harmonia Sacra Singings'
    @subtitle = '&nbsp;'
    @filename = 'minutes.html' 
  end 

  def gen_body
    gen_template('minutes_body.erb')
  end
end

class PageNumberIndexPage < HSPage
  def initialize
    super 
    @title = 'Page Number Index'
    @subtitle = 'Pages indexed by page number in the 26th edition'   
    @filename = 'page_index.html' 
  end 
  
  def mycmp(page1, page2)
   # puts "#{page1} vs. #{page2}"
    n1, p1 = split_page(page1)
    n2, p2 = split_page(page2)
    if n1==n2
      return -1 if p1=='t' 
      return 1
    else
      n1<=>n2
    end
  end
  def gen_body
    tunes = $tunes.sort{ |a,b| mycmp(a[Page],b[Page])} 
    @out.puts "<h2>Page Number</h2>"
    @out.puts '<p>"T" means top of page; "B" means bottom of page. '
    @out.puts "<ul>"
    tunes.each do |t|  
      n,p = split_page(t[Page]) 
      @out.puts "<li> <a href=\"#{t[Page]}.html\">#{n}#{p}</a> <a href=\"#{t[Page]}.html\">#{t[Tune]}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" if t[Tune] and t[Tune].size > 0
    end
    @out.puts "</ul>"
  end
end
 
class AuthorIndexPage < HSPage
  def initialize
    super 
    @title = 'Author Index'
    @subtitle = '&nbsp;'   
    @filename = 'author_index.html' 
  end 
  
  def gen_body
    authors = Hash.new
    tunes = $tunes 
    #puts "#{tunes.size}"
    tunes.each do |line|
      author = line[Author]
      if author  and author != ''
        authors[author] ||= []
        authors[author] << line
      end
    end
    author_list = authors.keys.sort 
    #puts "#{author_list.size}"
    @out.puts "<h2>Authors</h2><ul>"
    author_list.each  do |name|
      a_tunes = authors[name].sort{ |a,b| a[Tune].to_s <=> b[Tune].to_s}
      @out.puts "<li>#{name}"
      @out.puts "<ul>"
      a_tunes.each do |t|  
        n,p = split_page(t[Page]) 
        @out.puts "<li><a href=\"#{t[Page]}.html\">#{t[Tune]}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" 
      end
      @out.puts "</ul>" 
    end
    @out.puts "</ul>"
  end
end      

class ComposerIndexPage < HSPage
  def initialize
    super 
    @title = 'Composer Index'
    @subtitle = '&nbsp;'   
    @filename = 'composer_index.html' 
  end 
  
  def gen_body
    composers = Hash.new
    tunes = $tunes 
    tunes.each do |line| 
      composer = line[Composer] 
      if composer  and composer != ''
        composers[composer] ||= []
        composers[composer] << line
      end
    end
    composer_list = composers.keys.sort 
    @out.puts "<h2>Composers</h2><ul>"
    composer_list.each  do |name|
      a_tunes = composers[name].sort{ |a,b| a[Tune].to_s <=> b[Tune].to_s} 
      @out.puts "<li>#{name}"
      @out.puts "<ul>"
      a_tunes.each do |t|  
        n,p = split_page(t[Page]) 
        @out.puts "<li><a href=\"#{t[Page]}.html\">#{t[Tune]}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" 
      end
      @out.puts "</ul>" 
    end
    @out.puts "</ul>"
  end
end
            
class MetreIndexPage < HSPage
  def initialize
    super 
    @title = 'Meter Index'
    @subtitle = '&nbsp;'   
    @filename = 'meter_index.html' 
  end 

  def to_numbers(text)
    if text=='Various'
      [1/0.0]
    elsif text=='L.M. 8,8,8,8.'
      [0]
    elsif text=='C.M. 8,6,8,6.'
      [1]
    elsif text=='S.M. 6,6,8,6.'
      [2]
    else
      text.gsub(/[^,0123456789]/,'').split(/,/).map {|i| i.to_i}
    end
  end
  
  def mycmp(a,b)
    as = to_numbers(a)
    bs = to_numbers(b)
    l = [as.size, bs.size].min
    l.times do |i|
      return as[i] <=> bs[i] if as[i] != bs[i]
    end
    as.size <=> bs.size
  end
  
  def gen_body
    meters = Hash.new
    tunes = $tunes 
    #puts "#{tunes.size}"
    tunes.each do |line|
      meter = line[Metre]
      if meter  and meter != ''
        meters[meter] ||= []
        meters[meter] << line
      end
    end 
    #puts "#{meters.keys.inspect}"
    meter_list = meters.keys.sort{ |a,b| mycmp(a,b) }  
    #puts "#{meter_list.size}"
    @out.puts "<h2>Meters</h2><ul>"
    meter_list.each  do |name|
      a_tunes = meters[name].sort{ |a,b| a[Tune].to_s <=> b[Tune].to_s}
      @out.puts "<li>#{name}"
      @out.puts "<ul>"
      a_tunes.each do |t|  
        n,p = split_page(t[Page]) 
        @out.puts "<li><a href=\"#{t[Page]}.html\">#{t[Tune]}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" 
      end
      @out.puts "</ul>" 
    end
    @out.puts "</ul>"
  end
end

class IncipitIndexPage < HSPage
  def initialize
    super 
    @title = 'Incipit Index'
    @subtitle = '&nbsp;'   
    @filename = 'incipit_index.html' 
  end 
  
  def incipit_to_integer(str)
    case str[0..0].downcase
      when "#"
         -1
      when "d" 
         0
      when "r" 
         1
      when "m" 
         2
      when "f" 
         3
      when "s" 
         4
      when "l" 
         5
      when "t" 
         6
    else 
      raise "Invalid incipit: #{str}"
    end
  end
  
  def incipit_compare_string(s1, s2)
    #puts "#{s1}<=>#{s2}"
    s1.gsub(/ +/,'').split(//).map {|s| incipit_to_integer(s)}  <=>  s2.gsub(/ +/,'').split(//).map {|s| incipit_to_integer(s)}
  end
  
  def gen_body
    incipits = Hash.new
    tunes = $tunes 
    # puts "#{tunes.size}"
    tunes.each do |line|
      incipit = line[Doremi]
      if incipit  and incipit != ''
        incipits[incipit] ||= []
        incipits[incipit] << line
      end
    end
    incipit_list = incipits.keys.sort{ |a,b| incipit_compare_string(a,b) }   
    # puts "#{incipit_list.size}"
    @out.puts "<h2>Incipits</h2><p>Incipits are given using the first letter of Do Re Mi Fa So La Ti. </p><ul>"
    incipit_list.each  do |name|
      a_tunes = incipits[name].sort{ |a,b| a[Tune].to_s <=> b[Tune].to_s}
      @out.puts "<li><a href=\"#{a_tunes[0][Page]}.html\">#{name.strip}</a>" # silly.
      a_tunes.each do |t|  
        n,p = split_page(t[Page]) 
        @out.puts "<a href=\"#{t[Page]}.html\">#{t[Tune].strip}</a> <a href=\"#{t[Page]}.html\">#{n}#{p}</a>&nbsp;&nbsp;&nbsp;(#{t[Firstline][0..50]}...)</li>" 
      end
    end
    @out.puts "</ul>"
  end
end      

class TunePage < HSPage
  def initialize(tune,tunes,i)
    super()
    @prev_page = tunes[(i-1)%tunes.size][Page]
    @next_page = tunes[(i+1)%tunes.size][Page]
    @tune_name = tune[Tune]
    @tune_author = tune[Author]
    @tune_composer = tune[Composer]
    @tune_page = calc_fpage(tune[Page])
    @title = @tune_name
    @subtitle = calc_subtitle(tune[Page])  
    @filename = "#{tune[Page]}.html"
  end
  def next_page
    "#{@next_page}.html"
  end
  def prev_page
   "#{@prev_page}.html" 
  end   
  def calc_subtitle(pg) 
    subtitle = []
    if @tune_composer and @tune_composer.size > 0
      subtitle << "Tune: #{@tune_composer.split(/, +/).reverse.join(' ')}"
    end
    if @tune_author and @tune_author.size > 0
      subtitle << "Text: #{@tune_author.split(/, +/).reverse.join(' ')}"
    end 
    subtitle << "Harmonia Sacra, page #{pg}."    
    subtitle.join('. ')
  end
  def calc_fpage(str)
    n,p = split_page(str)
    if p=='t'
      "#{n}a"
    elsif p=='b'
      "#{n}b"
    else
      str
    end
  end
  def tune_image
    finess_page("hs_tune_images/#{@tune_page}.png")
  end
  def tune_4
    finess_page("hs_4_pdf/#{@tune_page}.pdf")
  end
  def tune_7
    finess_page("hs_7_pdf/#{@tune_page}.pdf")
  end
  def midi
    finess_page("midis/#{@tune_page}.MID")
  end
  def mp3
    finess_page("mp3s/#{@tune_page}.mp3")
  end
  def gen_body
    gen_template('tune_body.erb')
  end
end  

def make_site 
  HomePage.new.gen
  CopyrightPage.new.gen
  TitleIndexPage.new.gen
  AltTitleIndexPage.new.gen
  FirstIndexPage.new.gen
  PageNumberIndexPage.new.gen
  AuthorIndexPage.new.gen
  ComposerIndexPage.new.gen
  MetreIndexPage.new.gen
  IncipitIndexPage.new.gen    
  MinutesPage.new.gen 
  $tunes.each_with_index {|tn,i| TunePage.new(tn,$tunes,i).gen}.size
end 

## -- RAKE TASKS START HERE ...

desc "Create Home Page"
file 'index.html' => ['templates/welcome_body.erb', 'templates/footer.erb', 'templates/header.erb',
                      'templates/head.erb', 'templates/left.erb'] do |t| 
  HomePage.new.gen 
end 

desc "Create Copyright Page"
file 'copyright.html' => ['templates/copyright_body.erb', 'templates/footer.erb', 'templates/header.erb',
                      'templates/head.erb', 'templates/left.erb'] do |t| 
  CopyrightPage.new.gen 
end 

desc "Title Index Page"
file 'title_index.html' => ['templates/footer.erb', 'templates/header.erb',
            'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  TitleIndexPage.new.gen 
end 

desc "Alt Title Index Page"
file 'alt_title_index.html' => ['templates/footer.erb', 'templates/header.erb',
                                'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  AltTitleIndexPage.new.gen 
end

desc "Page Number Index Page"
file 'page_index.html' => ['templates/footer.erb', 'templates/header.erb',
                           'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  PageNumberIndexPage.new.gen 
end

desc "First Line Index Page"
file 'first_line_index.html' => ['templates/footer.erb', 'templates/header.erb',
                           'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  FirstIndexPage.new.gen 
end

desc "Author Index Page"
file 'author_index.html' => ['templates/footer.erb', 'templates/header.erb',
                             'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  AuthorIndexPage.new.gen 
end


desc "Composer Index Page"
file 'composer_index.html' => ['templates/footer.erb', 'templates/header.erb',
                              'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  ComposerIndexPage.new.gen 
end

desc "Metre Index Page"
file 'meter_index.html' => ['templates/footer.erb', 'templates/header.erb',
                            'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  MetreIndexPage.new.gen 
end

desc "Incipit Index Page"
file 'incipit_index.html' => ['templates/footer.erb', 'templates/header.erb',
                              'templates/head.erb', 'templates/left.erb', 'data/handbook.xml'] do |t| 
  IncipitIndexPage.new.gen 
end

desc "Minutes Page"
file 'minutes.html' => ['templates/minutes_body.erb', 'templates/footer.erb', 'templates/header.erb',
                        'templates/head.erb', 'templates/left.erb'] do |t| 
  MinutesPage.new.gen 
end


desc "Site"
task :site => ['alt_title_index.html', 
         'author_index.html', 
         'composer_index.html', 
         'copyright.html', 
         'first_line_index.html', 
         'incipit_index.html', 
         'index.html', 
         'meter_index.html', 
         'minutes.html', 
         'page_index.html', 
         'title_index.html',
         'templates/tune_body.erb', 
         'templates/footer.erb', 
         'templates/header.erb',
         'templates/head.erb', 
         'templates/left.erb', 
         'data/handbook.xml'] do |t| 
           $tunes.each_with_index {|tn,i| TunePage.new(tn,$tunes,i).gen}
 end
