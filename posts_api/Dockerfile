# syntax = docker/dockerfile:1

# Make sure RUBY_VERSION matches the Ruby version in .ruby-version and Gemfile
ARG RUBY_VERSION=3.2.2
FROM public.ecr.aws/docker/library/ruby:3.2.2-slim as base

# Rails app lives here
WORKDIR /rails

# Set production environment
ENV RAILS_ENV="production" \
    BUNDLE_DEPLOYMENT="1" \
    BUNDLE_PATH="/usr/local/bundle" \
    BUNDLE_WITHOUT="development" \
    SECRET_KEY_BASE="IiyMI0rqFZsJkw3+bfCoBCSEGOn18XUxEYYxveU0xI1flzxLnWkispJCCnrcl62rHFiZ9enMpEqAV2ZP1va0WSadxBtf0F0EX6dPOwsNz1VlDvzUF2JuWFVmwZybGybDxJR49cV4tXKpSN8LJi2Tr04ByRlWeUXNXK8sjYUQ5xoTLr/uaROw4o/bxd+c0Q+D+imh1ZWWEWVJt/9bSfT/Il2PQX4lohW5cbGHHdwI74gILfZug+JcmMcquozXLF/JUnigifI7e9uf9jfCh5dB6weraWCCPB94ezEY7UkhzSGovZ0bJSoI20/Z5DPrA8HOo+9N+2VDdeZcQfiQNMtTVu4BDGq4dOFdWYU7S2PtcK1vdbnft02xYTXCRxyITHsg+Z5ojYDMdy7IrCXttwxbt41H8MTw--R1HjUJQisogqd10T--xJUQNNGLid0sgUdnbS1JYQ=="\
    RAILS_MASTER_KEY="08fa6b366db1d512c9707207a4fd1d06"

# Throw-away build stage to reduce size of final image
FROM base as build

# Install packages needed to build gems
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential git libvips pkg-config

# Install application gems
COPY Gemfile Gemfile.lock ./
RUN bundle install && \
    rm -rf ~/.bundle/ "${BUNDLE_PATH}"/ruby/*/cache "${BUNDLE_PATH}"/ruby/*/bundler/gems/*/.git && \
    bundle exec bootsnap precompile --gemfile

# Copy application code
COPY . .

# Precompile bootsnap code for faster boot times
RUN bundle exec bootsnap precompile app/ lib/

# Adjust binfiles to be executable on Linux
RUN chmod +x bin/* && \
    sed -i "s/\r$//g" bin/* && \
    sed -i 's/ruby\.exe$/ruby/' bin/*


# Copy credentials and other necessary files
COPY config/credentials.yml.enc /rails/config/

# Final stage for app image
FROM base

# Install packages needed for deployment
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y curl libsqlite3-0 libvips && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Copy built artifacts: gems, application
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build /rails /rails

# Run and own only the runtime files as a non-root user for security
RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails db log storage tmp
USER rails:rails

# Entrypoint prepares the database.
ENTRYPOINT ["/rails/bin/docker-entrypoint"]

# Start the server by default, this can be overwritten at runtime
EXPOSE 3000
CMD ["./bin/rails", "server"]
