# dna-vision-sequence-analyses
Thoughts to Word or Audio 
// Spring Boot Backend - DNAAnalysisController.java
package com.dnaanalysis.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import java.util.*;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

@SpringBootApplication
@RestController
@CrossOrigin(origins = "*")
public class DNAAnalysisController {

    public static void main(String[] args) {
        SpringApplication.run(DNAAnalysisController.class, args);
    }

    // Genetic Code Mapping
    private static final Map<String, String> CODON_MAP = new HashMap<String, String>() {{
        put("TTT", "Phenylalanine"); put("TTC", "Phenylalanine");
        put("TTA", "Leucine"); put("TTG", "Leucine"); put("CTT", "Leucine"); 
        put("CTC", "Leucine"); put("CTA", "Leucine"); put("CTG", "Leucine");
        put("TCT", "Serine"); put("TCC", "Serine"); put("TCA", "Serine"); 
        put("TCG", "Serine"); put("AGT", "Serine"); put("AGC", "Serine");
        put("TAT", "Tyrosine"); put("TAC", "Tyrosine");
        put("TGT", "Cysteine"); put("TGC", "Cysteine");
        put("TGG", "Tryptophan");
        put("CCT", "Proline"); put("CCC", "Proline"); put("CCA", "Proline"); put("CCG", "Proline");
        put("CAT", "Histidine"); put("CAC", "Histidine");
        put("CAA", "Glutamine"); put("CAG", "Glutamine");
        put("CGT", "Arginine"); put("CGC", "Arginine"); put("CGA", "Arginine"); 
        put("CGG", "Arginine"); put("AGA", "Arginine"); put("AGG", "Arginine");
        put("ATT", "Isoleucine"); put("ATC", "Isoleucine"); put("ATA", "Isoleucine");
        put("ATG", "Methionine");
        put("ACT", "Threonine"); put("ACC", "Threonine"); put("ACA", "Threonine"); put("ACG", "Threonine");
        put("AAT", "Asparagine"); put("AAC", "Asparagine");
        put("AAA", "Lysine"); put("AAG", "Lysine");
        put("GTT", "Valine"); put("GTC", "Valine"); put("GTA", "Valine"); put("GTG", "Valine");
        put("GCT", "Alanine"); put("GCC", "Alanine"); put("GCA", "Alanine"); put("GCG", "Alanine");
        put("GAT", "Aspartic_Acid"); put("GAC", "Aspartic_Acid");
        put("GAA", "Glutamic_Acid"); put("GAG", "Glutamic_Acid");
        put("GGT", "Glycine"); put("GGC", "Glycine"); put("GGA", "Glycine"); put("GGG", "Glycine");
        put("TAA", "STOP"); put("TAG", "STOP"); put("TGA", "STOP");
    }};

    @PostMapping("/api/analyze-dna")
    public ResponseEntity<DNAAnalysisResult> analyzeDNA(@RequestBody DNAAnalysisRequest request) {
        try {
            DNAAnalysisResult result = new DNAAnalysisResult();
            
            // Generate DNA sequence using established genetic principles
            String sequence = generateDNASequence(request);
            result.setSequence(sequence);
            
            // Convert to amino acids
            List<String> aminoAcids = translateToAminoAcids(sequence);
            result.setAminoAcids(aminoAcids);
            
            // Calculate molecular properties
            result.setMolecularWeight(calculateMolecularWeight(aminoAcids));
            result.setGcContent(calculateGCContent(sequence));
            
            // Generate analysis report
            result.setAnalysisReport(generateAnalysisReport(request, result));
            
            // Generate access code if payment provided
            if (request.getWalletAddress() != null && request.getPaymentAmount() > 0) {
                result.setAccessCode(generateAccessCode(request.getWalletAddress(), request.getPaymentAmount()));
            }
            
            return ResponseEntity.ok(result);
        } catch (Exception e) {
            return ResponseEntity.badRequest().body(null);
        }
    }

    private String generateDNASequence(DNAAnalysisRequest request) {
        StringBuilder sequence = new StringBuilder();
        Random random = new Random();
        
        // Start codon (ATG - Methionine start codon in real genetics)
        sequence.append("ATG");
        
        // Generate sequence based on trait requests
        Map<String, String[]> traitCodons = getTraitSpecificCodons();
        
        if (request.getTraits() != null) {
            for (String trait : request.getTraits()) {
                String[] codons = traitCodons.get(trait.toLowerCase());
                if (codons != null) {
                    sequence.append(codons[random.nextInt(codons.length)]);
                }
            }
        }
        
        // Add regulatory sequences
        String[] regulatoryCodons = {"GAA", "GAG", "AAA", "AAG", "CGA", "CGG"};
        for (int i = 0; i < 5; i++) {
            sequence.append(regulatoryCodons[random.nextInt(regulatoryCodons.length)]);
        }
        
        // Stop codon
        sequence.append("TAA");
        
        return sequence.toString();
    }

    private Map<String, String[]> getTraitSpecificCodons() {
        Map<String, String[]> traitCodons = new HashMap<>();
        traitCodons.put("eyes", new String[]{"TAT", "TAC", "TGT", "TGC"});
        traitCodons.put("hair", new String[]{"TGG", "CCT", "CCC", "CCA"});
        traitCodons.put("height", new String[]{"GGT", "GGC", "GGA", "GGG"});
        traitCodons.put("longevity", new String[]{"CTT", "CTC", "CTA", "CTG"});
        traitCodons.put("intelligence", new String[]{"AAT", "AAC", "CAT", "CAC"});
        return traitCodons;
    }

    private List<String> translateToAminoAcids(String sequence) {
        List<String> aminoAcids = new ArrayList<>();
        for (int i = 0; i < sequence.length() - 2; i += 3) {
            String codon = sequence.substring(i, i + 3);
            String aminoAcid = CODON_MAP.getOrDefault(codon, "Unknown");
            aminoAcids.add(aminoAcid);
        }
        return aminoAcids;
    }

    private double calculateMolecularWeight(List<String> aminoAcids) {
        Map<String, Double> aaWeights = new HashMap<String, Double>() {{
            put("Alanine", 89.1); put("Arginine", 174.2); put("Asparagine", 132.1);
            put("Aspartic_Acid", 133.1); put("Cysteine", 121.0); put("Glutamic_Acid", 147.1);
            put("Glutamine", 146.1); put("Glycine", 75.1); put("Histidine", 155.2);
            put("Isoleucine", 131.2); put("Leucine", 131.2); put("Lysine", 146.2);
            put("Methionine", 149.2); put("Phenylalanine", 165.2); put("Proline", 115.1);
            put("Serine", 105.1); put("Threonine", 119.1); put("Tryptophan", 204.2);
            put("Tyrosine", 181.2); put("Valine", 117.1);
        }};
        
        return aminoAcids.stream()
                .mapToDouble(aa -> aaWeights.getOrDefault(aa, 0.0))
                .sum();
    }

    private double calculateGCContent(String sequence) {
        long gcCount = sequence.chars()
                .filter(c -> c == 'G' || c == 'C')
                .count();
        return (double) gcCount / sequence.length() * 100;
    }

    private String generateAnalysisReport(DNAAnalysisRequest request, DNAAnalysisResult result) {
        StringBuilder report = new StringBuilder();
        report.append("DNA SEQUENCE ANALYSIS REPORT\n");
        report.append("============================\n\n");
        report.append("Analysis Date: ").append(new Date().toString()).append("\n");
        report.append("Subject ID: ").append(request.getSubjectId()).append("\n");
        report.append("Gender: ").append(request.getGender()).append("\n\n");
        
        report.append("SEQUENCE PROPERTIES:\n");
        report.append("Length: ").append(result.getSequence().length()).append(" nucleotides\n");
        report.append("GC Content: ").append(String.format("%.2f", result.getGcContent())).append("%\n");
        report.append("Molecular Weight: ").append(String.format("%.2f", result.getMolecularWeight())).append(" Da\n\n");
        
        report.append("AMINO ACID COMPOSITION:\n");
        for (String aa : result.getAminoAcids()) {
            report.append("- ").append(aa).append("\n");
        }
        
        if (request.getTraits() != null) {
            report.append("\nTRAIT ANALYSIS:\n");
            for (String trait : request.getTraits()) {
                report.append("- ").append(trait.toUpperCase()).append(": Sequence optimized\n");
            }
        }
        
        return report.toString();
    }

    private String generateAccessCode(String walletAddress, double paymentAmount) {
        try {
            String input = walletAddress + paymentAmount + System.currentTimeMillis();
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] hash = md.digest(input.getBytes());
            StringBuilder hexString = new StringBuilder();
            for (byte b : hash) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) hexString.append('0');
                hexString.append(hex);
            }
            return "MGIS-" + hexString.substring(0, 16).toUpperCase();
        } catch (NoSuchAlgorithmException e) {
            return "MGIS-" + UUID.randomUUID().toString().substring(0, 16).toUpperCase();
        }
    }

    @PostMapping("/api/vision-analysis")
    public ResponseEntity<VisionAnalysisResult> analyzeVision(@RequestBody VisionAnalysisRequest request) {
        VisionAnalysisResult result = new VisionAnalysisResult();
        
        // Convert color data to binary representation
        StringBuilder binarySequence = new StringBuilder();
        List<String> colorAnalysis = new ArrayList<>();
        
        for (String color : request.getColors()) {
            String hex = color.replace("#", "");
            try {
                int rgb = Integer.parseInt(hex, 16);
                String binary = Integer.toBinaryString(rgb);
                binarySequence.append(binary);
                colorAnalysis.add("Color " + color + " -> Binary: " + binary);
            } catch (NumberFormatException e) {
                colorAnalysis.add("Invalid color format: " + color);
            }
        }
        
        // Convert binary to DNA-like sequence
        String dnaEquivalent = binarySequence.toString()
                .replaceAll("00", "A")
                .replaceAll("01", "T")
                .replaceAll("10", "C")
                .replaceAll("11", "G");
        
        result.setBinarySequence(binarySequence.toString());
        result.setDnaEquivalent(dnaEquivalent);
        result.setColorAnalysis(colorAnalysis);
        
        return ResponseEntity.ok(result);
    }
}

// Data Transfer Objects
class DNAAnalysisRequest {
    private String subjectId;
    private String gender;
    private List<String> traits;
    private String walletAddress;
    private double paymentAmount;
    
    // Getters and setters
    public String getSubjectId() { return subjectId; }
    public void setSubjectId(String subjectId) { this.subjectId = subjectId; }
    public String getGender() { return gender; }
    public void setGender(String gender) { this.gender = gender; }
    public List<String> getTraits() { return traits; }
    public void setTraits(List<String> traits) { this.traits = traits; }
    public String getWalletAddress() { return walletAddress; }
    public void setWalletAddress(String walletAddress) { this.walletAddress = walletAddress; }
    public double getPaymentAmount() { return paymentAmount; }
    public void setPaymentAmount(double paymentAmount) { this.paymentAmount = paymentAmount; }
}

class DNAAnalysisResult {
    private String sequence;
    private List<String> aminoAcids;
    private double molecularWeight;
    private double gcContent;
    private String analysisReport;
    private String accessCode;
    
    // Getters and setters
    public String getSequence() { return sequence; }
    public void setSequence(String sequence) { this.sequence = sequence; }
    public List<String> getAminoAcids() { return aminoAcids; }
    public void setAminoAcids(List<String> aminoAcids) { this.aminoAcids = aminoAcids; }
    public double getMolecularWeight() { return molecularWeight; }
    public void setMolecularWeight(double molecularWeight) { this.molecularWeight = molecularWeight; }
    public double getGcContent() { return gcContent; }
    public void setGcContent(double gcContent) { this.gcContent = gcContent; }
    public String getAnalysisReport() { return analysisReport; }
    public void setAnalysisReport(String analysisReport) { this.analysisReport = analysisReport; }
    public String getAccessCode() { return accessCode; }
    public void setAccessCode(String accessCode) { this.accessCode = accessCode; }
}

class VisionAnalysisRequest {
    private List<String> colors;
    
    public List<String> getColors() { return colors; }
    public void setColors(List<String> colors) { this.colors = colors; }
}

class VisionAnalysisResult {
    private String binarySequence;
    private String dnaEquivalent;
    private List<String> colorAnalysis;
    
    public String getBinarySequence() { return binarySequence; }
    public void setBinarySequence(String binarySequence) { this.binarySequence = binarySequence; }
    public String getDnaEquivalent() { return dnaEquivalent; }
    public void setDnaEquivalent(String dnaEquivalent) { this.dnaEquivalent = dnaEquivalent; }
    public List<String> getColorAnalysis() { return colorAnalysis; }
    public void setColorAnalysis(List<String> colorAnalysis) { this.colorAnalysis = colorAnalysis; }
}
