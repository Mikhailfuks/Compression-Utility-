#include <iostream>
#include <fstream>
#include <vector>
#include <zlib.h>

void compressFile(const std::string& inputFile, const std::string& outputFile) {
    // Open the input file
    std::ifstream input(inputFile, std::ios::binary);
    if (!input.is_open()) {
        std::cerr << "Could not open input file: " << inputFile << std::endl;
        return;
    }

    // Read the entire input file into a vector
    std::vector<char> buffer((std::istreambuf_iterator<char>(input)), std::istreambuf_iterator<char>());
    input.close();

    // Prepare for compression
    uLong sourceLen = buffer.size();
    uLong destLen = compressBound(sourceLen); // Allocate enough space
    std::vector<char> compressedData(destLen);

    int result = compress(reinterpret_cast<Bytef *>(compressedData.data()), &destLen,
                          reinterpret_cast<const Bytef *>(buffer.data()), sourceLen);
    if (result != Z_OK) {
        std::cerr << "Compression failed, error code: " << result << std::endl;
        return;
    }

    // Write the compressed data to the output file
    std::ofstream output(outputFile, std::ios::binary);
    if (!output.is_open()) {
        std::cerr << "Could not open output file: " << outputFile << std::endl;
        return;
    }

    // Write the compressed size followed by the compressed data
    output.write(reinterpret_cast<const char*>(&destLen), sizeof(destLen));
    output.write(compressedData.data(), destLen);
    output.close();

    std::cout << "File compressed successfully: " << outputFile << std::endl;
}

void decompressFile(const std::string& inputFile, const std::string& outputFile) {
    // Open the compressed input file
    std::ifstream input(inputFile, std::ios::binary);
    if (!input.is_open()) {
        std::cerr << "Could not open input file: " << inputFile << std::endl;
        return;
    }

    // Read the compressed size
    uLong destLen = 0;
    input.read(reinterpret_cast<char*>(&destLen), sizeof(destLen));

    // Read the compressed data
    std::vector<char> compressedData(destLen);
    input.read(compressedData.data(), destLen);
    input.close();

    // Prepare for decompression
    std::vector<char> buffer(destLen * 3); // Initial size (just an estimation)
    uLong sourceLen = compressedData.size();
    
    int result = uncompress(reinterpret_cast<Bytef *>(buffer.data()), &destLen,
                            reinterpret_cast<const Bytef *>(compressedData.data()), sourceLen);
    if (result != Z_OK) {
        std::cerr << "Decompression failed, error code: " << result << std::endl;
        return;
    }

    // Write the decompressed data to the output file
    std::ofstream output(outputFile, std::ios::binary);
    if (!output.is_open()) {
        std::cerr << "Could not open output file: " << outputFile << std::endl;
        return;
    }

    output.write(buffer.data(), destLen);
    output.close();

    std::cout << "File decompressed successfully: " << outputFile << std::endl;
}



int main(int argc, char* argv[]) {
    if (argc < 4) {
        std::cerr << "Usage: " << argv[0] << " <compress|decompress> <input_file> <output_file>" << std::endl;
        return 1;
    }

    std::string mode = argv[1];
    std::string inputFile = argv[2];
    std::string outputFile = argv[3];

    if (mode == "compress") {
        compressFile(inputFile, outputFile);
    } else if (mode == "decompress") {
        decompressFile(inputFile, outputFile);
    } else {
        std::cerr << "Invalid mode. Use 'compress' or 'decompress'." << std::endl;
        return 1;
    }

    return 0;
}
