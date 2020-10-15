require 'rake'
require 'docker'

MARKDOWN_RESUME_IMAGE_NAME = 'there4/markdown-resume'
RESUME_TYPES = %w[
  pdf
  html
]

namespace :resume do
  task :clean do |_t, args|
    dirs = [
      'tmp',
      'assets/resumes'
    ]

    dirs.each do |dir|
      dir_name = File.join(Dir.pwd, dir)
      unless File.directory?(dir_name)
        FileUtils.mkdir_p(dir_name)
      end
    end
  end

  # remove jekyll related content and write to tmp/ dir
  task :write_temp_resume do |_t, args|
    f = File.read('resume.md')
    resume = f.partition("<!-- JEKYLL-REMOVE-END -->").last

    File.write('tmp/resume.md', resume)
  end

  task :pull_image do |_t, args|
    puts "Pulling image, #{MARKDOWN_RESUME_IMAGE_NAME} ..."
    Docker::Image.create(
      'fromImage' => MARKDOWN_RESUME_IMAGE_NAME,
    )
  end

  desc 'Build resume from markdown. Example - rake resume:build[html]'
  task :build, [:type] => [:clean, :write_temp_resume, :pull_image] do |_t, args|
    type = args[:type] || 'pdf'

    unless RESUME_TYPES.include?(type)
      raise "#{type} is not a supported resume type, try one of these: #{RESUME_TYPES.join(',')}"
    end

    puts "Building resume from markdown file ..."
    container = Docker::Container.create(
      'Cmd' => [
        'md2resume',
        type,
        'tmp/resume.md',
        'assets/resumes/'
      ],
      'Image' => MARKDOWN_RESUME_IMAGE_NAME,
      'Volumes' => {
        Dir.pwd => { '/resume' => 'rw' },
      },
      'HostConfig' => {
        'Binds' => ["#{Dir.pwd}:/resume"],
      },
    )

    container.start
    result = container.wait['StatusCode']

    if !result.zero?
      raise "Got bad status code when building resume: #{result}"
    end

    puts "Deleting container ..."
    container.delete(:force => true)

    puts "Done. You can find your #{type} resume in /assets/resumes"
  end
end

