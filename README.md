#include <iostream>
#include <boost/asio.hpp>
#include <string>
#include <thread>

using boost::asio::ip::tcp;

class DynamicWebServer {
public:
    DynamicWebServer(boost::asio::io_context& io_context, short port)
        : acceptor_(io_context, tcp::endpoint(tcp::v4(), port)) {
        start_accept();
    }

private:
    void start_accept() {
        tcp::socket socket(acceptor_.get_executor().context());
        acceptor_.async_accept(socket,
            [this, &socket](boost::system::error_code ec) {
                if (!ec) {
                    std::string response = handle_request();
                    boost::asio::write(socket, boost::asio::buffer(response));
                }
                start_accept();
            });
    }

    std::string handle_request() {
        // Contoh konten dinamis
        std::string content = "<html><body><h1>Dynamic Web Server</h1>";
        content += "<p>Current time: " + std::to_string(std::time(nullptr)) + "</p>";
        content += "</body></html>";

        std::string response = "HTTP/1.1 200 OK\r\nContent-Length: " + std::to_string(content.size()) + "\r\n\r\n" + content;
        return response;
    }

    tcp::acceptor acceptor_;
};

int main() {
    try {
        boost::asio::io_context io_context;
        DynamicWebServer server(io_context, 8080);
        io_context.run();
    }
    catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }

    return 0;
}
