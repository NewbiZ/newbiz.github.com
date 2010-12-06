---
layout: page_post
title: Welcome again
category: Personal
---
Subtitle 1
----------
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi laoreet vehicula magna,
at consequat nisi lobortis nec. Aenean vulputate semper arcu sed auctor. Nunc tempor
vulputate varius. Sed tristique placerat justo eu consequat. Nulla convallis fringilla
magna, vel bibendum nibh dapibus eget. Vestibulum eget eleifend lacus. Suspendisse
consectetur euismod interdum. Phasellus ut tellus dui, vitae vehicula leo. Suspendisse
potenti. Integer eu eleifend lorem. Nullam ante turpis, blandit at cursus nec, scelerisque
sed odio. Phasellus id dolor turpis, nec mollis velit. Donec erat ipsum, aliquet id ornare
ultricies, facilisis at nulla. Morbi ut lectus sem, in sagittis ante. Vestibulum molestie
sollicitudin est eu imperdiet. Nam malesuada scelerisque diam eget rutrum. Curabitur tempor
euismod augue malesuada auctor. Vivamus sit amet urna vitae enim ullamcorper aliquet. Mauris
fringilla porta dolor, eget consequat sapien aliquam quis.

Nullam sollicitudin libero et magna tempus quis volutpat erat egestas. Donec cursus porttitor
lectus nec commodo. Vivamus accumsan est in nisi interdum eget sollicitudin urna lobortis.
Morbi gravida, felis eu fermentum adipiscing, eros quam ultricies nisl, quis tristique orci
ligula at dui. Mauris laoreet tortor at est suscipit vel consectetur felis euismod. In ante
tellus, suscipit id dictum et, pharetra quis quam. Ut iaculis pharetra tortor in vehicula.
Morbi ultrices diam vel purus tincidunt euismod. Pellentesque bibendum sollicitudin urna. Nunc
id massa massa. Nullam lobortis nunc in tellus bibendum elementum scelerisque turpis tristique.
Nunc ac vestibulum dui.

{% highlight ruby %}
#!/usr/bin/env ruby

require 'FileUtils'
require 'optparse'
require 'pp'

options = {
  :verbose => false,
  :noop    => false,
  :project => ''
}
optparse = OptionParser.new do |opt|
  ### Parameters
  opt.program_name = "project"
  opt.version      = [ 1, 0, 0 ]
  opt.banner       = "Usage: #{opt.program_name} [options] project_name"
  opt.define_head 'Generate a CMake/C++ project tree'
  
  ### Common options
  opt.separator 'Common:'
  opt.on( '-p', '--project project_name', String, 'Project name' ) { |p| options[:project]=p }
  opt.on( '-n', '--[no-]noop', 'Don\'t create files' ) { |n| options[:noop]=n }
  
  ### Misc options
  opt.separator 'Misc:'
  opt.on( '-v', '--[no-]verbose', 'Display information about the status of the generation' ) { |v| options[:verbose]=v }
  opt.on_tail( '-h', '--help', 'Display this help screen' ) { puts opt; exit }
  opt.on_tail( '--version', 'Display the version of this script' ) { puts "This is #{opt.program_name} v#{opt.version.join('.')}"; exit }
  
  ### Parsing
  begin
    opt.parse!
  rescue OptionParser::InvalidOption => e
    puts e
    puts opt.banner
    puts 'Use --help to display list of available options'
    exit
  end
end
{% endhighlight %}

Subtitle 2
----------
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi laoreet vehicula magna,
at consequat nisi lobortis nec. Aenean vulputate semper arcu sed auctor. Nunc tempor
vulputate varius. Sed tristique placerat justo eu consequat. Nulla convallis fringilla
magna, vel bibendum nibh dapibus eget. Vestibulum eget eleifend lacus. Suspendisse
consectetur euismod interdum. Phasellus ut tellus dui, vitae vehicula leo. Suspendisse
potenti. Integer eu eleifend lorem. Nullam ante turpis, blandit at cursus nec, scelerisque
sed odio. Phasellus id dolor turpis, nec mollis velit. Donec erat ipsum, aliquet id ornare
ultricies, facilisis at nulla. Morbi ut lectus sem, in sagittis ante. Vestibulum molestie
sollicitudin est eu imperdiet. Nam malesuada scelerisque diam eget rutrum. Curabitur tempor
euismod augue malesuada auctor. Vivamus sit amet urna vitae enim ullamcorper aliquet. Mauris
fringilla porta dolor, eget consequat sapien aliquam quis.

This is a title
===============

Subtitle 1
----------
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi laoreet vehicula magna,
at consequat nisi lobortis nec. Aenean vulputate semper arcu sed auctor. Nunc tempor
vulputate varius. Sed tristique placerat justo eu consequat. Nulla convallis fringilla
magna, vel bibendum nibh dapibus eget. Vestibulum eget eleifend lacus. Suspendisse
consectetur euismod interdum. Phasellus ut tellus dui, vitae vehicula leo. Suspendisse
potenti. Integer eu eleifend lorem. Nullam ante turpis, blandit at cursus nec, scelerisque
sed odio. Phasellus id dolor turpis, nec mollis velit. Donec erat ipsum, aliquet id ornare
ultricies, facilisis at nulla. Morbi ut lectus sem, in sagittis ante. Vestibulum molestie
sollicitudin est eu imperdiet. Nam malesuada scelerisque diam eget rutrum. Curabitur tempor
euismod augue malesuada auctor. Vivamus sit amet urna vitae enim ullamcorper aliquet. Mauris
fringilla porta dolor, eget consequat sapien aliquam quis.

Nullam sollicitudin libero et magna tempus quis volutpat erat egestas. Donec cursus porttitor
lectus nec commodo. Vivamus accumsan est in nisi interdum eget sollicitudin urna lobortis.
Morbi gravida, felis eu fermentum adipiscing, eros quam ultricies nisl, quis tristique orci
ligula at dui. Mauris laoreet tortor at est suscipit vel consectetur felis euismod. In ante
tellus, suscipit id dictum et, pharetra quis quam. Ut iaculis pharetra tortor in vehicula.
Morbi ultrices diam vel purus tincidunt euismod. Pellentesque bibendum sollicitudin urna. Nunc
id massa massa. Nullam lobortis nunc in tellus bibendum elementum scelerisque turpis tristique.
Nunc ac vestibulum dui.

Subtitle 2
----------
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi laoreet vehicula magna,
at consequat nisi lobortis nec. Aenean vulputate semper arcu sed auctor. Nunc tempor
vulputate varius. Sed tristique placerat justo eu consequat. Nulla convallis fringilla
magna, vel bibendum nibh dapibus eget. Vestibulum eget eleifend lacus. Suspendisse
consectetur euismod interdum. Phasellus ut tellus dui, vitae vehicula leo. Suspendisse
potenti. Integer eu eleifend lorem. Nullam ante turpis, blandit at cursus nec, scelerisque
sed odio. Phasellus id dolor turpis, nec mollis velit. Donec erat ipsum, aliquet id ornare
ultricies, facilisis at nulla. Morbi ut lectus sem, in sagittis ante. Vestibulum molestie
sollicitudin est eu imperdiet. Nam malesuada scelerisque diam eget rutrum. Curabitur tempor
euismod augue malesuada auctor. Vivamus sit amet urna vitae enim ullamcorper aliquet. Mauris
fringilla porta dolor, eget consequat sapien aliquam quis.
