require 'rake/clean'
require 'rake/tasklib'

# see https://ruby.github.io/rake/doc/rakefile_rdoc.html for guidance on Rakefile format

DOC_FILES = FileList.new(File.join('docs', '**', '*.md'))
SPEC_FILES = FileList.new(File.join('reference', '*.yaml'))

TEMP_DIR = 'temp'
DIST_DIR = 'dist'

CLEAN.add(FileList.new(File.join(TEMP_DIR, '**', '*')))
CLOBBER.add(FileList.new(File.join(DIST_DIR, '**', '*')))

######################################################################
# Helper methods & classes
######################################################################

def bundled(spec)
    File.join(TEMP_DIR, spec.to_s.pathmap('%f'))
end

def folder(package, format, spec = nil)
    File.join(DIST_DIR, package.to_s, format.to_s, spec.nil? ? '' : spec.to_s.pathmap('%n'))
end

class GeneratorTask < Rake::TaskLib
    def generate
        fail NotImplementedError
    end
end

class OpenAPIGeneratorTask < GeneratorTask
    TOOL = 'openapi-generator-cli'

    def initialize(generator = nil)
        @generator = generator
    end

    def generate(package, format, source_file, target_dir)
        target_dir = File.join(target_dir, source_file.pathmap('%n'))
        directory target_dir

        # desc "Generate #{target} from #{source} using generator '#{self}'"
        log_file = File.join(target_dir, 'openapi-generator.log')
        file log_file => [ source_file, target_dir ] do
            sh "#{TOOL} generate --input-spec #{source_file} --generator-name #{@generator} --output #{target_dir} > #{log_file}"
        end

        desc "Generate '#{format}' #{package} using generator '#{TOOL}/#{@generator}'"
        task format => [ log_file ]
    end
end

class OpenAPICodeGenTask < GeneratorTask
    TOOL = 'oapi-codegen'

    def generate(package, format, source_file, target_dir)
        @package = package
        @format = format
        @source_file = source_file

        target_dir = File.join(target_dir, source_file.pathmap('%n'))
        directory target_dir

        codegen('server', File.join(target_dir, 'api', 'gen_server.go'))
        codegen('spec', File.join(target_dir, 'api', 'gen_spec.go'))
        codegen('types', File.join(target_dir, 'api', 'gen_types.go'))
        codegen('types', File.join(target_dir, 'client', 'types.go'))
        codegen('client', File.join(target_dir, 'client', 'client.go'))
    end

    private
    def codegen(type, target_file)
        target_dir = File.dirname(target_file)
        directory target_dir

        file target_file => [ @source_file, target_dir ] do
            sh "#{TOOL} -exclude-tags=confirmation -package=api -generate=#{type} -o #{target_file} #{source_file}"
        end

        desc "Generate '#{@format}' #{@package} using generator '#{TOOL}'"
        task @format => [ target_file ]
    end
end

class PostManGeneratorTask < GeneratorTask
    TOOL = 'openapi2postmanv2'

    def generate(package, format, source_file, target_dir)
        directory target_dir

        target_file = File.join(target_dir, source_file.to_s.pathmap('%n.json'))
        file target_file => [ source_file, target_dir ] do
            sh "#{TOOL} --spec #{source_file} --output #{target_file} --pretty"
        end

        desc "Generate '#{format}' #{package} using generator '#{TOOL}'"
        task format => [ target_file ]
    end
end

######################################################################
# Tasks
######################################################################

task :default => %w[ all ]

desc "Perform all tasks"
task :all =>  %w[ validate generate ]

desc "Install tool dependencies"
task :install do
    sh "npm install -g markdownlint-cli"
    sh "npm install -g @apidevtools/swagger-cli"
    sh "npm install -g @stoplight/spectral"
    sh "npm install -g @openapitools/openapi-generator-cli"
    sh "go get github.com/deepmap/oapi-codegen/cmd/oapi-codegen"
    sh "npm install -g openapi-to-postmanv2"
end

desc "Test tool dependencies"
task :selftest do
    sh "markdownlint --version"
    sh "swagger-cli --version"
    sh "spectral --version"
    sh "openapi-generator-cli version"
    sh "oapi-codegen -version"
    sh "openapi2postmanv2 --version"
end

desc "Show pending TODOs"
task :pending do
    sh "grep -R -i TODO * | grep -v Rakefile"
end

namespace :bundle do
    directory TEMP_DIR

    SPEC_FILES.each do |spec|
        bundled_spec = bundled(spec)
        # desc "Bundle spec '#{spec}' with Swagger CLI (swagger-cli)"
        file bundled_spec => [ spec, TEMP_DIR ] do
            sh "swagger-cli bundle --dereference --type yaml --outfile #{bundled_spec} #{spec}"
        end

        desc "Bundle specs (some tools don't handle $ref well)"
        task :specs => [ bundled_spec ]
    end
end

task :validate => %w[ validate:all ]

namespace :validate do
    namespace :docs do
        DOC_FILES.each do |doc|
            desc "Validate docs with Markdownlint"
            task :markdownlint => [ doc ] do
                sh "markdownlint #{doc}"
            end
        end

        desc "Validate docs with all tools"
        task :all => %w[ markdownlint ]
    end

    namespace :specs do
        SPEC_FILES.each do |spec|
            desc "Validate specs with Stoplight Spectral (spectral)"
            task :spectral => [ spec ] do
                sh "spectral lint --verbose #{spec}"
            end

            desc "Validate specs with Swagger CLI (swagger-cli)"
            task :swagger_cli => [ spec ] do
                sh "swagger-cli validate #{spec}"
            end

            bundled_spec = bundled(spec)
            desc "Validate specs with OpenAPI Generator (openapi-generator-cli)"
            task :openapi_generator => [ bundled_spec ] do
                sh "openapi-generator-cli validate --input-spec #{bundled_spec}"
            end
        end

        desc "Validate specs with all tools"
        task :all => %w[ spectral swagger_cli openapi_generator ]
    end

    desc "Validate all files"
    task :all => %w[ docs:all specs:all ]
end

task :generate => %w[ generate:all ]

namespace :generate do
    directory DIST_DIR

    PACKAGES = {
        docs: {
            html: OpenAPIGeneratorTask.new('html2'),
            markdown: OpenAPIGeneratorTask.new('markdown'),
        },
        client: {
            javascript: OpenAPIGeneratorTask.new('javascript'),
            typescript: OpenAPIGeneratorTask.new('typescript-axios'),
            # both 'python' and 'python-legacy' generators barf on the clinic spec
            # python: OpenAPIGeneratorTask.new('python-legacy'),
            swift: OpenAPIGeneratorTask.new('swift5'),
            ruby: OpenAPIGeneratorTask.new('ruby'),
            lua: OpenAPIGeneratorTask.new('lua'),
            go: OpenAPIGeneratorTask.new('go'),
        },
        server: {
            go: OpenAPICodeGenTask.new(),
        },
        test: {
            postman: PostManGeneratorTask.new()
        }
    }

    PACKAGES.each do |package, formats|
        namespace package do
            formats.each do |format, generator|
                SPEC_FILES.each do |spec|
                    target_dir = folder(package, format)
                    generator.generate(package, format, bundled(spec), target_dir)
                end

                desc "Generate #{package} for all formats: #{formats.keys.map { |l| l.to_s }.join(', ')}"
                task :all => [ format, DIST_DIR ]
            end
        end

        desc "Generate all of #{PACKAGES.keys.map { |p| p.to_s }.join(', ')}"
        task :all => [ "#{package}:all" ]
    end
end
