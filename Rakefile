# -*- ruby -*-

#
# This is a Ruby file, used by the "rake" make-like program.
#

begin
  # First, we use a few ruby things...
  require 'rubygems'
  require 'rubygems/package_task'
end
require 'rdoc/task'
require 'rake/testtask'
require 'shellwords'

#
# The default task is run if rake is given no explicit arguments.
#
desc "Default Task"
task :default => :test

#
# Test tasks
#
Rake::TestTask.new do |t|
  t.libs << 'test'
  t.pattern = 'test/test*.rb'
  t.verbose = true
end


def unreleasable_reason
  can_release_package
  unreleasable_reason
end

def can_release_package
  reasons = []

  unless news_is_current
    reasons << 'the NEWS file is not current'
  end

  unless defined?(Gem)
    reasons << 'RubyGems is not installed'
  end

  reason = if reasons.empty?
             ""
           else
             last = reasons.pop
             ("Can not release package because " +
              (reasons.empty? ? "#{last}." :
               (reasons.join(", ") + " and #{last}.")))
           end
  can_release = reason.length == 0

  self.class.module_eval <<-END_OF_CODE
  def unreleasable_reason
    \"#{reason}\"
  end
  def can_release_package
    #{can_release.inspect}
  end
  END_OF_CODE

  can_release_package
end

# Is the NEWS file current?
def news_is_current
  today = Time.now.strftime('%Y-%m-%d')
  version = Regexp.new(Regexp.quote(PKG_VERSION))
  if IO.readlines('NEWS').first =~
      /= Changes in RubyMail #{PKG_VERSION} \(released #{today}\)$/
    true
  else
    false
  end
end


#
# These PKG_ variables are used by Rake's package rule.
#
PKG_VERSION = begin
                version= IO.readlines('version').first.chomp
                if version =~ /^\d+\.\d+\.\d+$/
                  version.untaint
                else
                  fail "package version is bogus"
                end
                version
              end

PKG_FILES = FileList.new('test/**/*',
                         'guide/**/*',
                         'lib/**/*',
                         'install.rb',
                         'NEWS',
                         'NOTES',
                         'README',
                         'THANKS',
                         'TODO',
                         'Rakefile',
                         'version')

#
# Teach Rake how to build the RDoc documentation for this package.
#
rdoc = Rake::RDocTask.new do |rdoc|
  rdoc.main = 'README'
  rdoc.rdoc_files.include("README", "NEWS", "THANKS",
                          "TODO", "guide/*.txt", "lib/**/*.rb")
  rdoc.rdoc_files.exclude(/\bSCCS\b/,
                          "lib/rubymail/parser/*")
  unreleased = if can_release_package
                 ""
               else
                 " (UNRELEASED!)"
               end
  rdoc.title = "RubyMail Documentation (version #{PKG_VERSION})"
  rdoc.options << '--exclude' << 'SCCS'
end

# Make sure that we don't package anything that hasn't been tagged.
task :package => [ :can_release ]

desc "Check if the package is in a releasable state."
task :can_release do
  unless can_release_package
    puts unreleasable_reason
  end
end

#
# Create a Gem::Specification right in the Rakefile, using some of the
# variables we have set up above.
#
if defined?(Gem)
  spec = Gem::Specification.new do |s|
    s.name = 'rmail-sup'
    s.version = PKG_VERSION + if can_release_package
                                ''
                              else
                                '.666'
                              end
    s.summary = 'A MIME mail parsing and generation library.'
    s.description = <<-EOF
    RMail is a lightweight mail library containing various utility classes and
    modules that allow ruby scripts to parse, modify, and generate MIME mail
    messages.
    EOF

    s.files = PKG_FILES.to_a

    s.required_ruby_version = Gem::Version::Requirement.new(">= 1.8.1")

    s.has_rdoc = true
    s.extra_rdoc_files = rdoc.rdoc_files.reject { |fn| fn =~ /\.rb$/ }.to_a
    s.rdoc_options.concat([ '--title', rdoc.title, '--main', rdoc.main,
                            rdoc.options ].flatten)

    s.test_files = FileList['test/tc_*.rb'].to_a

    s.author = "Sup maintainers"
    s.email = "sup-devel@rubyforge.org"
    s.homepage = "http://supmua.org"
  end

  #
  # Use our Gem::Specification to make some package tasks.
  #
  Gem::PackageTask.new(spec) do |pkg|
    pkg.need_zip = true
    pkg.need_tar = true
  end
end

desc "Install RubyMail using the standard install.rb script"
task :install do
  ruby "install.rb"
end

